# Avaliação Preliminar e Classificação de Risco — Inteligência Artificial

**ID do Documento:** RC-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical — Módulo Aura+ (IA Conversacional)

**Norma:** Resolução CFM 2.454/2026 Arts. 12–13 e Anexo II

---

## 1. Propósito

Este documento formaliza a avaliação preliminar de risco do módulo Aura+ conforme exigido pelo Art. 12 da Resolução CFM 2.454/2026, que determina que toda instituição médica que desenvolver ou utilizar IA deverá realizar avaliação preliminar para definir o grau de risco da solução.

## 2. Metodologia de Avaliação

Conforme Art. 12, parágrafo único, a avaliação considera os seguintes fatores:

| **Fator** | **Avaliação para Aura+** |
|---|---|
| Potencial impacto nos direitos fundamentais e saúde | Moderado — processa dados sensíveis de saúde, mas não executa ações clínicas autônomas. |
| Criticidade do contexto de uso | Moderado — apoio ao bem-estar e longevidade, não emergência ou UTI. |
| Complexidade e grau de autonomia do modelo | Alto — LLM generativo (Claude, Anthropic), mas saídas são informativas, não prescritivas. |
| Finalidade pretendida | Insights personalizados de saúde e bem-estar; suporte informacional. |
| Finalidades potenciais (uso indevido) | Paciente pode interpretar saídas como diagnóstico — mitigado por disclaimers e Safety Gate. |
| Nível de intervenção humana no resultado | Alto — Doctor-in-the-Loop para protocolos; Safety Gate para crises; IA não prescreve. |
| Quantidade e sensibilidade dos dados | Alta — dados de saúde (labs, wearables, PHQ-9, GAD-7) classificados como sensíveis pela LGPD. |

## 3. Classificação de Risco

### 3.1 Resultado: MÉDIO RISCO

Com base nos critérios do Anexo II da Resolução CFM 2.454/2026, o módulo Aura+ é classificado como **solução de médio risco**.

**Justificativa (Anexo II, item II):**

> *"Aplicações de IA cujo uso envolve algum potencial de impacto adverso, porém mitigável via supervisão humana ativa e controles de segurança. Enquadram-se aqui sistemas que apoiam decisões clínicas ou operacionais importantes, mas não as executam de forma autônoma, de modo que erros do algoritmo possam ser detectados e corrigidos pelo profissional antes de causarem dano."*

**Enquadramento da Aura+:**

- A Aura+ **apoia** o utilizador com insights de saúde baseados em seus dados, mas **não executa** diagnósticos, prescrições ou ações clínicas autônomas.
- Erros do LLM são mitigados pelo **Safety Gate** (HAZ-01, HAZ-02, HAZ-03) que intercepta crises antes do LLM, e pelo **Doctor-in-the-Loop** que bloqueia protocolos de intervenção sem validação médica.
- A supervisão humana está ativa em todas as camadas críticas do sistema.

### 3.2 Exclusão de Alto Risco

A Aura+ **não** se enquadra como alto risco porque:

- Não influencia diretamente decisões médicas críticas — o sistema explicitamente declara que não diagnostica nem prescreve (CONSENT.md §4.1.7).
- Não executa ações automatizadas com consequências clínicas — protocolos permanecem bloqueados até validação por médico.
- Não opera em contextos de pacientes em estado vulnerável (UTI, emergência) — foco em longevidade e bem-estar preventivo.

### 3.3 Exclusão de Baixo Risco

A Aura+ **não** se enquadra como baixo risco porque:

- Processa dados pessoais sensíveis de saúde (labs, wearables, instrumentos clínicos).
- Gera conteúdo personalizado que pode influenciar comportamentos de saúde do utilizador.
- Utiliza IA generativa (LLM), cuja natureza probabilística introduz risco de informações imprecisas.

## 4. Obrigações Decorrentes da Classificação (Médio Risco)

Conforme Anexo II, §2°, as soluções de médio risco exigem:

| **Obrigação** | **Implementação na Aura Medical** | **Documento** |
|---|---|---|
| Monitoramento regular | Auditoria mensal do `ai_audit_log` | AI_AUDIT.md §5.1 |
| Avaliação de desempenho/viés em intervalos apropriados | Monitoramento trimestral de viés | BIAS_MONITORING.md |
| Reavaliação se houver aumento de criticidade | Gatilhos de reclassificação definidos (Seção 5) | Este documento |
| Controles proporcionais de segurança | SHA-256 audit, RLS, consent gate, Safety Gate | AI_AUDIT.md, CONSENT.md |

## 5. Gatilhos de Reclassificação

A classificação de médio risco deverá ser **reavaliada** nos seguintes cenários:

### 5.1 Gatilhos para Reclassificação a Alto Risco

- A Aura+ passar a emitir recomendações terapêuticas específicas sem mediação médica.
- O Doctor-in-the-Loop for removido ou enfraquecido.
- O sistema passar a operar em contextos de emergência ou UTI.
- A IA passar a influenciar diretamente dosagens, prescrições ou diagnósticos.
- Incidentes de segurança do paciente atribuíveis a saídas da IA (HAZ-01 a HAZ-04 materializados).

### 5.2 Gatilhos para Reclassificação a Baixo Risco

- A Aura+ deixar de processar dados pessoais sensíveis de saúde.
- O sistema passar a fornecer apenas informações genéricas (não personalizadas).
- A IA generativa for substituída por sistema baseado em regras sem personalização.

## 6. Comunicação ao Utilizador (Art. 13)

Conforme Art. 13, o nível de risco deve ser **informado ao usuário**. Esta informação é apresentada:

- Na tela de consentimento do Aura+ (CONSENT.md §4.1), com a declaração: *"Este módulo utiliza Inteligência Artificial classificada como médio risco conforme Resolução CFM 2.454/2026."*
- Na política de privacidade acessível no app.

## 7. Periodicidade de Revisão

| **Evento** | **Ação** |
|---|---|
| A cada 12 meses | Revisão ordinária desta classificação. |
| Mudança funcional significativa no Aura+ | Reavaliação imediata conforme Seção 5. |
| Publicação de diretrizes complementares do CFM | Revisão de conformidade em até 30 dias. |
| Incidente de segurança do paciente | Reavaliação imediata conforme Seção 5.1. |

## 8. Rastreabilidade

| **Requisito** | **Vínculo** |
|---|---|
| Avaliação Preliminar de Risco (Art. 12) | Este documento |
| Categorização de Risco (Art. 13, Anexo II) | Seção 3 |
| Obrigações de Médio Risco (Anexo II, §2°) | Seção 4 |
| Comunicação ao Utilizador (Art. 13) | Seção 6 |
| Safety Gate (HAZ-01 a HAZ-04) | RISK_ANALYSIS.md |
| Doctor-in-the-Loop | RISK_ANALYSIS.md, CONSENT.md |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
