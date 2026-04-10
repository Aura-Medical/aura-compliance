# Política de Auditoria de Inteligência Artificial

**ID do Documento:** AA-001

**Revisão:** 1.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical — Módulo Aura+ (IA Conversacional)

**Normas:** CFM Resolução 2.314/2022 Art. 5 (Transparência em IA Médica), LGPD Art. 6, VII (Princípio da Segurança)

---

## 1. Propósito

Este documento define as regras de auditoria e rastreabilidade para todas as interações de Inteligência Artificial (IA) realizadas pelo módulo Aura+ do sistema Aura Medical. O objetivo é garantir que toda decisão de suporte clínico assistida por LLM seja reconstruível e auditável, em conformidade com o Art. 5 da Resolução CFM 2.314/2022 sobre utilização de IA em saúde.

## 2. Princípio Fundamental

> **Toda interação entre um utilizador e o LLM deve ser rastreável em sua totalidade, sem que dados pessoais sensíveis (PHI) sejam armazenados em texto claro no log de auditoria.**

Este princípio concilia duas obrigações aparentemente conflitantes:
- **CFM 2.314 Art. 5:** Exige auditabilidade completa das decisões assistidas por IA.
- **LGPD Art. 11:** Restringe o tratamento de dados pessoais sensíveis ao mínimo necessário.

A solução adotada é o **hashing criptográfico (SHA-256)** do conteúdo das interações.

## 3. Arquitetura de Logging

### 3.1 Tabela de Auditoria (`ai_audit_log`)

Cada interação com o LLM gera um registro na tabela `ai_audit_log` do Supabase com os seguintes campos:

| **Campo** | **Tipo** | **Descrição** | **Contém PHI?** |
|---|---|---|---|
| `id` | UUID | Identificador único do registro. | Não |
| `user_id` | UUID | Referência ao utilizador (FK para `auth.users`). | Sim (identificador) |
| `session_id` | UUID | Identificador da sessão de chat. | Não |
| `model` | TEXT | Versão exata do modelo LLM (snapshot datado). | Não |
| `prompt_hash` | TEXT | Hash SHA-256 do prompt completo enviado ao LLM. | Não |
| `response_hash` | TEXT | Hash SHA-256 da resposta completa recebida do LLM. | Não |
| `token_count` | INTEGER | Número de tokens consumidos na interação. | Não |
| `is_crisis_bypass` | BOOLEAN | Indica se o Safety Gate foi acionado (bypass do LLM). | Não |
| `created_at` | TIMESTAMPTZ | Timestamp da interação (UTC). | Não |

### 3.2 Processo de Hashing

```
Input: prompt_completo (system_prompt + user_message + snapshot_context)
Output: SHA-256(prompt_completo) → prompt_hash

Input: resposta_completa (todos os blocos SSE concatenados)
Output: SHA-256(resposta_completa) → response_hash
```

**Código-Fonte:** `aura-backend/src/health/aura-plus/audit.ts`

**Função:** `logAiInteraction()`

**Biblioteca:** `crypto.createHash('sha256')` (Node.js nativo)

### 3.3 Garantia de Integridade

- O hash SHA-256 é **unidirecional** — não é possível reconstruir o prompt ou resposta original a partir do hash.
- Em caso de auditoria regulatória, o prompt pode ser **reconstruído** localmente a partir do snapshot do utilizador e do system prompt versionado, e seu hash comparado com o registro armazenado para provar correspondência exata.
- A coluna `model` armazena o **snapshot datado** do LLM (ex: `claude-haiku-4-5-20251001`), impedindo que atualizações silenciosas da API alterem o comportamento sem rastreabilidade (REQ-21).

## 4. Regras de Operação

### 4.1 Logging Obrigatório

Todo acesso ao LLM DEVE gerar um registro de auditoria. Não há exceções.

| **Tipo de Interação** | **Logging** | **Campos Especiais** |
|---|---|---|
| Chat normal (Aura+) | Obrigatório | `prompt_hash`, `response_hash`, `model` |
| Opener (mensagem inicial) | Obrigatório | `prompt_hash`, `response_hash`, `model` |
| Crisis Bypass (Safety Gate) | Obrigatório | `is_crisis_bypass = true`, `model = 'safety-bypass'` |
| Consent Denied (403) | Não aplicável | Nenhum dado trafega — log não necessário |

### 4.2 Design Fail-Soft

O logging de auditoria utiliza um design **fail-soft**:

- Se a gravação do log falhar (ex: timeout do Supabase), a resposta ao utilizador **NÃO é bloqueada**.
- A falha de logging é registrada internamente para retry.
- **Racional:** Bloquear a resposta ao utilizador por uma falha de logging criaria um risco clínico (HAZ-12) maior do que a perda temporária de um registro de auditoria.

### 4.3 Proibição de Log Bruto

É **estritamente proibido** armazenar o conteúdo bruto (não-hashado) de prompts ou respostas na tabela de auditoria ou em qualquer outro log do sistema. Esta regra existe para:

- Cumprir o princípio da minimização de dados (LGPD Art. 6, III).
- Evitar que uma violação do banco de dados (`RISK_ANALYSIS.md` HAZ-08) exponha o conteúdo das conversas de saúde.
- Garantir que o `ai_audit_log` possa ser compartilhado com auditores regulatórios sem violar a privacidade dos titulares.

### 4.4 Versão do Modelo — Rastreabilidade Exata

Para fins de auditoria (CFM 2.314), é **proibido** o uso de aliases genéricos para o LLM (ex: `claude-haiku` sem data). A versão do modelo deve ser fixada em um snapshot datado:

- **Constante:** `HAIKU_MODEL` em `aura-backend/src/health/aura-plus/anthropic.ts`
- **Valor Atual:** `claude-haiku-4-5-20251001`
- **Regra:** Qualquer atualização de versão do modelo exige aprovação do RT e atualização do registro em `CONFIG_MGMT.md`.

## 5. Procedimento de Auditoria

### 5.1 Auditoria de Rotina (Mensal)

1. Consultar `ai_audit_log` para verificar:
   - Volume de interações por utilizador/sessão.
   - Taxa de acionamento do Safety Gate (`is_crisis_bypass = true`).
   - Consistência da coluna `model` (todas as linhas devem conter o snapshot datado atual).

2. Verificar que nenhuma coluna contém texto claro de prompt/resposta.

### 5.2 Auditoria Regulatória (Sob Demanda)

Em caso de solicitação da ANPD, CFM ou Anvisa:

1. **Reconstrução de Prompt:** Recriar o prompt a partir do `health_snapshot_cache` do utilizador na data da interação + versão do `system-prompt.ts` na tag Git correspondente.
2. **Verificação de Hash:** Comparar o SHA-256 do prompt reconstruído com o `prompt_hash` armazenado.
3. **Prova de Correspondência:** Se os hashes coincidem, está provado que o prompt registrado foi exatamente o enviado ao LLM.

## 6. Retenção de Dados

| **Artefato** | **Período de Retenção** | **Racional** |
|---|---|---|
| `ai_audit_log` | 5 anos | Alinhado com prazos de retenção de prontuário (CFM Res. 1.821/2007). |
| Tags Git (system-prompt) | Permanente | Necessário para reconstrução de prompt em auditoria. |
| `health_snapshot_cache` | 6 horas (TTL ativo) | Dados voláteis; reconstruíveis a partir das tabelas-fonte. |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
