# Gestão de Consentimento para Processamento por LLM

**ID do Documento:** CN-001

**Revisão:** 1.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical — Módulo Aura+ (IA Conversacional)

**Normas:** LGPD (Lei nº 13.709/2018), Art. 7, I e Art. 11, I — Consentimento para Dados Pessoais Sensíveis; Resolução CFM 2.454/2026 Arts. 4°–6°, 10°–11°

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

### 2.2 Relação com a Resolução CFM 2.454/2026

A Resolução CFM 2.454/2026 estabelece requisitos amplos sobre consentimento, informação ao paciente e direitos no contexto de IA médica:

| **Dispositivo** | **Obrigação** | **Implementação na Aura Medical** |
|---|---|---|
| Art. 4°, V | Registrar no prontuário o uso de IA como apoio à decisão. | Registro em `ai_audit_log` vinculado ao `user_id` e `session_id` (§3.1 de AI_AUDIT.md). |
| Art. 5°, §1 | Informar o paciente quando IA é utilizada como apoio relevante. | Tela de consentimento dedicada (§3.1 deste documento). |
| Art. 5°, §2 | Vedado delegar à IA a comunicação de diagnósticos, prognósticos ou decisões terapêuticas. | A Aura+ explicitamente não diagnostica nem prescreve; protocolos requerem validação médica (§2.3). |
| Art. 5°, §3 | Respeitar a recusa informada do paciente ao uso de IA. | Consent gate fail-closed + direito de recusa (§2.4). |
| Art. 10°, II | Direito à obtenção de segunda opinião. | Aura+ não substitui consulta médica; disclaimers explícitos. |
| Art. 10°, IV | Direito à privacidade e confidencialidade dos dados. | Hashing SHA-256 de interações, proibição de log bruto (AI_AUDIT.md §4.3). |
| Art. 11° | Utilização de IA deve ser comunicada e explicada aos pacientes. | Texto de consentimento (§4.1) + explicabilidade em AI_AUDIT.md §6.1. |

O consentimento da Aura Medical vai além dos requisitos mínimos, pois:

- Identifica explicitamente o **subprocessador** (Anthropic, Inc.) e o modelo utilizado.
- Informa que os dados são processados em servidores fora do Brasil (cloud).
- Garante que o consentimento é **separado** dos termos de uso gerais.
- Informa a **classificação de risco** da IA conforme RISK_CLASSIFICATION.md.

### 2.3 Vedação de Delegação de Diagnóstico à IA (Art. 5°, §2)

A Resolução CFM 2.454/2026 veda expressamente que o médico delegue à IA a comunicação de diagnósticos, prognósticos ou decisões terapêuticas sem mediação humana. A Aura+ atende a esta vedação por design:

- O system prompt contém instrução explícita de que a IA **não diagnostica doenças nem prescreve medicamentos**.
- Protocolos de intervenção permanecem **bloqueados** até validação por médico (Doctor-in-the-Loop).
- O Safety Gate intercepta situações de crise **antes** do LLM, encaminhando para serviços de emergência humanos.
- O texto de consentimento (§4.1.7) comunica esta limitação ao utilizador.

### 2.4 Direito de Recusa Informada do Uso de IA (Art. 5°, §3)

O paciente tem direito de recusar o uso de IA no seu cuidado. A Aura Medical implementa este direito:

- O consent gate (§3.3) opera em modo **fail-closed**: sem consentimento explícito, nenhum dado trafega para o LLM.
- A opção "Não, obrigado" na tela de consentimento (§3.1) permite recusa sem qualquer penalidade funcional — os demais recursos do app (avaliação de domínios, instrumentos clínicos, dados de wearables) permanecem disponíveis.
- A revogação de consentimento (§5) pode ser feita a qualquer momento em Configurações > Privacidade.
- A recusa ou revogação é registrada na tabela `user_consents` para fins de auditoria.

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
| `consent_category` | TEXT | Categoria de consentimento: `aura_plus_llm`. |
| `granted` | BOOLEAN | `true` se o utilizador concedeu; `false` se revogou. |
| `granted_at` | TIMESTAMPTZ | Timestamp da concessão ou revogação (UTC). |
| `ip_address` | INET | Endereço IP no momento da concessão/revogação. |
| `user_agent` | TEXT | User Agent do dispositivo no momento da ação. |
| `consent_text_version` | TEXT | Versão do texto de consentimento exibido (ex: `1.0`). |

### 3.3 Gate de Verificação (Fail-Closed)

**Código-Fonte:** `aura-backend/src/health/aura-plus/consent-gate.ts`

**Função:** `hasAuraPlusLlmConsent(userId: string)`

**Comportamento:**

1. Consulta a tabela `user_consents` filtrando por `user_id` e `consent_category = 'aura_plus_llm'`.
2. Verifica se `granted = true` na row mais recente (ordenada por `granted_at DESC`).
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
8. **Classificação de Risco (CFM 2.454/2026 Art. 13):** "Este módulo utiliza Inteligência Artificial classificada como médio risco conforme Resolução CFM 2.454/2026."
9. **Direito de Recusa (CFM 2.454/2026 Art. 5°, §3):** "Você tem o direito de recusar o uso de Inteligência Artificial no seu cuidado. A recusa não afeta o acesso aos demais recursos do aplicativo."
10. **Registro em Prontuário (CFM 2.454/2026 Art. 4°, V):** "O uso da IA como apoio é registrado em seu histórico para fins de rastreabilidade e auditoria."

### 4.2 Requisitos de Apresentação

- O texto deve estar em **PT-BR** claro e acessível.
- O botão de consentimento deve ser uma ação **afirmativa** (não uma caixa pré-marcada).
- O consentimento para Aura+ deve estar **separado** dos termos de uso e da política de privacidade gerais.
- O utilizador deve poder ler o texto completo **sem scroll forçado** (ou com scroll que atinja o final antes do botão ser habilitado).

## 5. Revogação de Consentimento

### 5.1 Processo de Revogação

1. Utilizador navega para **Configurações > Privacidade > Aura+ IA**.
2. Desativa o toggle "Permitir processamento por IA".
3. Backend insere nova row em `user_consents` com `granted = false` e `granted_at = NOW()`, preservando o histórico completo de consentimento.
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

1. A `consent_text_version` é incrementada.
2. Utilizadores com a versão anterior são apresentados novamente à tela de consentimento na próxima abertura do Aura+.
3. O consentimento anterior **não é invalidado automaticamente** — o utilizador recebe a opção de reconsentir ou revogar.

## 7. Rastreabilidade

| **Requisito** | **Vínculo** |
|---|---|
| Gate de Consentimento (LGPD Art. 11) | REQ-35 (`TRACEABILITY.md`) |
| PHI Exposta ao LLM sem Consentimento | HAZ-05 (`RISK_ANALYSIS.md`) |
| Retenção Indevida de Dados | HAZ-13 (`RISK_ANALYSIS.md`) |
| Teste de Consentimento | AI-04 (`VERIFICATION.md`) |
| Vedação de Delegação de Diagnóstico (CFM 2.454/2026 Art. 5°, §2) | §2.3 deste documento |
| Direito de Recusa Informada (CFM 2.454/2026 Art. 5°, §3) | §2.4 deste documento |
| Registro em Prontuário (CFM 2.454/2026 Art. 4°, V) | §2.2 deste documento, AI_AUDIT.md |
| Classificação de Risco comunicada (CFM 2.454/2026 Art. 13) | RISK_CLASSIFICATION.md, §4.1.8 deste documento |
| Explicabilidade (CFM 2.454/2026 Anexo I, XIX) | AI_AUDIT.md §6.1 |
| Contestabilidade (CFM 2.454/2026 Anexo I, XX) | AI_AUDIT.md §6.2 |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
