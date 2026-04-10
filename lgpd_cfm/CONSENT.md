# Gestão de Consentimento para Processamento por LLM

**ID do Documento:** CN-001

**Revisão:** 1.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical — Módulo Aura+ (IA Conversacional)

**Norma:** LGPD (Lei nº 13.709/2018), Art. 7, I e Art. 11, I — Consentimento para Dados Pessoais Sensíveis

---

## 1. Propósito

Este documento define a política de consentimento explícito e opt-in para o processamento de dados pessoais sensíveis de saúde (PHI) pelo Large Language Model (LLM) integrado ao módulo Aura+ do sistema Aura Medical. O documento garante conformidade com a LGPD Art. 11, que exige consentimento específico e destacado para tratamento de dados sensíveis.

## 2. Base Legal

### 2.1 Fundamentação LGPD

| **Artigo** | **Disposição** | **Aplicação na Aura Medical** |
|---|---|---|
| **Art. 5, II** | Dados pessoais sensíveis incluem dados relativos à saúde. | Todos os dados processados pela Aura Medical são classificados como sensíveis. |
| **Art. 7, I** | Tratamento mediante consentimento do titular. | Base legal primária para o módulo Aura+. |
| **Art. 8, §1** | Consentimento deve ser fornecido por escrito ou meio que demonstre a manifestação de vontade. | Consentimento digital auditado com timestamp, IP e User Agent. |
| **Art. 8, §4** | Consentimento deve referir-se a finalidades determinadas. | Consentimento específico para "processamento por IA conversacional" (não genérico). |
| **Art. 11, I** | Dados sensíveis: consentimento específico e em destaque. | Tela de consentimento dedicada, separada dos termos gerais. |
| **Art. 18, IX** | Direito de revogar o consentimento a qualquer momento. | Revogação disponível em Configurações > Privacidade. |

### 2.2 Relação com CFM 2.314/2022

A Resolução CFM 2.314/2022 exige que o paciente seja informado quando IA é utilizada no suporte à decisão clínica. O consentimento da Aura Medical vai além deste requisito, pois:

- Identifica explicitamente o **subprocessador** (Anthropic, Inc.) e o modelo utilizado.
- Informa que os dados são processados em servidores fora do Brasil (cloud).
- Garante que o consentimento é **separado** dos termos de uso gerais.

## 3. Arquitetura de Consentimento

### 3.1 Fluxo de Opt-In

```
[Utilizador tenta abrir Aura+ pela primeira vez]
        ↓
[Tela de Consentimento Dedicada]
  - Explica: dados de saúde serão processados por IA (Anthropic)
  - Explica: dados são hashados para auditoria (AI_AUDIT.md)
  - Explica: Doctor-in-the-Loop ativo para protocolos
  - Botão: "Concordo e Quero Usar a Aura+" (opt-in explícito)
  - Link: "Não, obrigado" (volta para app principal)
        ↓
[Backend registra consentimento na tabela `user_consents`]
        ↓
[Aura+ Chat liberado]
```

### 3.2 Tabela de Consentimento (`user_consents`)

| **Campo** | **Tipo** | **Descrição** |
|---|---|---|
| `id` | UUID | Identificador único do registro. |
| `user_id` | UUID | Referência ao utilizador (FK para `auth.users`). |
| `consent_type` | TEXT | Tipo de consentimento: `aura_plus_llm`. |
| `granted` | BOOLEAN | `true` se o utilizador concedeu; `false` se revogou. |
| `granted_at` | TIMESTAMPTZ | Timestamp da concessão (UTC). |
| `revoked_at` | TIMESTAMPTZ | Timestamp da revogação, se aplicável. |
| `ip_address` | TEXT | Endereço IP no momento da concessão/revogação. |
| `user_agent` | TEXT | User Agent do dispositivo no momento da ação. |
| `consent_version` | TEXT | Versão do texto de consentimento exibido (ex: `1.0`). |

### 3.3 Gate de Verificação (Fail-Closed)

**Código-Fonte:** `aura-backend/src/health/aura-plus/consent-gate.ts`

**Função:** `hasAuraPlusLlmConsent(userId: string)`

**Comportamento:**

1. Consulta a tabela `user_consents` filtrando por `user_id` e `consent_type = 'aura_plus_llm'`.
2. Verifica se `granted = true` e `revoked_at IS NULL`.
3. Se o consentimento **não existir** ou estiver **revogado**: retorna `false` → Backend responde **403 `consent_required`**.
4. Se o consentimento **existir** e estiver **ativo**: retorna `true` → Chat prossegue normalmente.

**Design Fail-Closed:** Na ausência de um registro de consentimento, o sistema **bloqueia** o acesso. Nunca assume consentimento implícito. Esta é uma mitigação direta para HAZ-05 (PHI exposta ao LLM sem consentimento).

## 4. Requisitos do Texto de Consentimento

O texto exibido ao utilizador na tela de consentimento DEVE conter:

### 4.1 Elementos Obrigatórios

1. **Identificação do Controlador:** Auramedical Tecnologia Ltda.
2. **Finalidade Específica:** "Processar seus dados de saúde através de Inteligência Artificial conversacional para fornecer insights personalizados de longevidade e bem-estar."
3. **Identificação do Subprocessador:** "Seus dados serão processados pela Anthropic, Inc. (EUA) através do modelo Claude."
4. **Dados Processados:** "Dados do seu perfil de saúde, resultados laboratoriais, dados de wearables e suas mensagens no chat."
5. **Garantias de Segurança:** "Suas interações são registradas com hash criptográfico (SHA-256) para fins de auditoria, sem armazenamento do conteúdo bruto das conversas."
6. **Direito de Revogação:** "Você pode revogar este consentimento a qualquer momento em Configurações > Privacidade. A revogação não afeta o uso dos demais recursos do app."
7. **Limitação do Sistema:** "A Aura+ não diagnostica doenças nem prescreve medicamentos. Protocolos de intervenção permanecem bloqueados até validação por um médico."

### 4.2 Requisitos de Apresentação

- O texto deve estar em **PT-BR** claro e acessível.
- O botão de consentimento deve ser uma ação **afirmativa** (não uma caixa pré-marcada).
- O consentimento para Aura+ deve estar **separado** dos termos de uso e da política de privacidade gerais.
- O utilizador deve poder ler o texto completo **sem scroll forçado** (ou com scroll que atinja o final antes do botão ser habilitado).

## 5. Revogação de Consentimento

### 5.1 Processo de Revogação

1. Utilizador navega para **Configurações > Privacidade > Aura+ IA**.
2. Desativa o toggle "Permitir processamento por IA".
3. Backend atualiza `user_consents`: define `granted = false` e `revoked_at = NOW()`.
4. A partir deste momento, `hasAuraPlusLlmConsent` retorna `false` → Aura+ Chat bloqueado.

### 5.2 Efeitos da Revogação

| **Funcionalidade** | **Disponível após Revogação?** |
|---|---|
| Avaliação de Domínios (DomainEvaluator) | Sim — funciona localmente sem LLM |
| Instrumentos Clínicos (PHQ-9, GAD-7, etc.) | Sim — cálculo local |
| Chat Aura+ (IA Conversacional) | **Não** — bloqueado pelo consent gate |
| Dados de Wearables e Labs | Sim — persistência local inalterada |
| Protocolos de Intervenção (Preview) | **Não** — dependem do chat Aura+ |

### 5.3 Preservação de Dados

A revogação do consentimento **não exclui** dados anteriormente processados ou registros de auditoria (`ai_audit_log`). Para exclusão completa, o utilizador deve solicitar a **Exclusão de Conta** (REQ-17), que aciona o procedimento de soft-delete em cascata.

## 6. Versionamento do Consentimento

Quando o texto de consentimento for atualizado (ex: mudança de subprocessador, adição de nova finalidade):

1. A `consent_version` é incrementada.
2. Utilizadores com a versão anterior são apresentados novamente à tela de consentimento na próxima abertura do Aura+.
3. O consentimento anterior **não é invalidado automaticamente** — o utilizador recebe a opção de reconsentir ou revogar.

## 7. Rastreabilidade

| **Requisito** | **Vínculo** |
|---|---|
| Gate de Consentimento (LGPD Art. 11) | REQ-35 (`TRACEABILITY.md`) |
| PHI Exposta ao LLM sem Consentimento | HAZ-05 (`RISK_ANALYSIS.md`) |
| Retenção Indevida de Dados | HAZ-13 (`RISK_ANALYSIS.md`) |
| Teste de Consentimento | AI-04 (`VERIFICATION.md`) |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
