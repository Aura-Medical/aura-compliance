# Procedimento Operacional Padrão — Resposta a Incidentes de Vazamento de Dados

**ID do Documento:** BR-001

**Revisão:** 1.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend

**Normas:** LGPD (Lei nº 13.709/2018), Arts. 46–49 — Segurança e Boas Práticas; Resolução CFM 2.454/2026 Arts. 6°–7°, 16°–17°

---

## 1. Propósito

Este documento estabelece o Procedimento Operacional Padrão (SOP) para identificação, contenção, notificação e remediação de incidentes de segurança envolvendo dados pessoais sensíveis de saúde (PHI) processados pelo sistema Aura Medical, em conformidade com a Lei Geral de Proteção de Dados (LGPD).

## 2. Escopo

Este SOP aplica-se a qualquer evento que resulte em:

- Acesso não autorizado a dados pessoais ou dados pessoais sensíveis de saúde.
- Divulgação acidental de PHI a terceiros.
- Perda ou destruição não intencional de registros de saúde.
- Comprometimento de chaves criptográficas, tokens de sessão ou credenciais de serviço.
- Falha nos controles de segurança definidos em `RISK_ANALYSIS.md` (HAZ-05 a HAZ-10, HAZ-13, HAZ-15).

## 3. Definições

| **Termo** | **Definição** |
|---|---|
| **Incidente de Segurança** | Qualquer evento confirmado ou suspeito que comprometa a confidencialidade, integridade ou disponibilidade de dados pessoais. |
| **PHI** | Dados Pessoais Sensíveis de Saúde conforme LGPD Art. 5, II — dados relativos à saúde. |
| **ANPD** | Autoridade Nacional de Proteção de Dados — órgão regulador da LGPD. |
| **Controlador** | Auramedical Tecnologia Ltda. — pessoa jurídica responsável pelas decisões de tratamento de dados. |
| **Encarregado (DPO)** | Frederico — responsável por comunicações com a ANPD e titulares. |

## 4. Equipe de Resposta a Incidentes

| **Função** | **Responsável** | **Responsabilidade** |
|---|---|---|
| Líder de Incidente | Arthur Teixeira de Almeida | Coordenação geral, decisões de contenção e comunicação executiva. |
| Encarregado (DPO) | Frederico | Notificação à ANPD e aos titulares; documentação regulatória. |
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | Avaliação do impacto clínico; determinação se a falha afeta a segurança do paciente. |
| Engenharia | Equipe de Desenvolvimento | Contenção técnica, análise forense e correção. |

## 5. Procedimento de Resposta (Fases)

### 5.1 Fase 1 — Identificação (0–2 horas)

**Objetivo:** Confirmar se um incidente de segurança ocorreu.

1. **Detecção:** O incidente pode ser identificado via:
   - Alertas automatizados do Supabase (tentativas de acesso anômalo, falha de RLS).
   - Notificação de um utilizador ou funcionário.
   - Auditoria interna dos logs de `ai_audit_log`.
   - Monitoramento de Certificate Pinning (falhas de handshake).

2. **Triagem Inicial:** O Líder de Incidente classifica a severidade:

| **Severidade** | **Critério** | **Exemplo** |
|---|---|---|
| **Crítica** | PHI de múltiplos utilizadores exposta; Safety Gate comprometido. | Vazamento da tabela `health_snapshot_cache`; falha no `isCrisisInput`. |
| **Alta** | PHI de um utilizador exposta; credencial comprometida. | Token JWT vazado; chave de serviço Supabase exposta. |
| **Média** | Dados não-sensíveis expostos; tentativa de acesso sem sucesso. | Email vazado sem dados de saúde; brute-force bloqueado. |
| **Baixa** | Vulnerabilidade identificada sem exploração confirmada. | Dependência com CVE publicado, sem evidência de exploit. |

3. **Registro:** Abrir registro de incidente com timestamp, descrição e classificação inicial.

### 5.2 Fase 2 — Contenção (2–24 horas)

**Objetivo:** Limitar o impacto do incidente.

**Ações Imediatas (por severidade):**

- **Crítica/Alta:**
  - Revogar tokens de sessão afetados via Supabase Auth.
  - Rotacionar chaves de serviço comprometidas.
  - Se o Safety Gate (`safety.ts`) estiver comprometido: ativar modo de manutenção no backend, desabilitando o chat Aura+ até correção.
  - Isolar registros afetados no banco de dados.

- **Média/Baixa:**
  - Aplicar patch de segurança na dependência afetada.
  - Reforçar monitoramento nos endpoints afetados.

### 5.3 Fase 3 — Notificação (até 72 horas)

**Obrigação Legal (LGPD Art. 48):** O controlador deverá comunicar à ANPD e ao titular a ocorrência de incidente de segurança que possa acarretar risco ou dano relevante aos titulares, em **prazo razoável** (recomendação ANPD: até 72 horas).

**3.1 Notificação à ANPD:**

O Encarregado (DPO) deve preparar comunicação contendo:

1. Descrição da natureza dos dados pessoais afetados.
2. Informações sobre os titulares envolvidos (sem identificação).
3. Indicação das medidas técnicas e de segurança utilizadas (RLS, Keychain, Certificate Pinning).
4. Os riscos relacionados ao incidente.
5. As medidas que foram ou que serão adotadas para reverter ou mitigar os efeitos.

**3.2 Notificação aos Titulares:**

- Comunicação clara e acessível em PT-BR, via email cadastrado.
- Deve incluir: o que aconteceu, quais dados foram afetados, o que a Aura Medical está fazendo, e orientações de proteção ao titular.
- **Prazo:** Simultâneo à notificação da ANPD.

### 5.4 Fase 4 — Erradicação e Recuperação (24–72 horas)

1. **Análise de Causa Raiz:** Documentar a cadeia de eventos que levou ao incidente.
2. **Correção:** Implementar fix no código-fonte, seguindo o processo de Change Control (CONFIG_MGMT.md Seção 4, categoria "Emergência").
3. **Verificação:** Executar suíte de testes de segurança (VERIFICATION.md Seção 4.4) para confirmar que a vulnerabilidade foi eliminada.
4. **Restauração:** Restaurar serviços normais com monitoramento intensificado por 30 dias.

### 5.5 Fase 5 — Pós-Incidente (7–30 dias)

1. **Relatório Final:** Documentar todo o incidente com timeline, ações tomadas, impacto confirmado e lições aprendidas.
2. **Atualização do FMEA:** Se o incidente revelar um hazard não previsto, atualizar `RISK_ANALYSIS.md` com novo HAZ-XX e mitigação correspondente.
3. **Revisão de Controles:** Avaliar se novos REQs são necessários em `TRACEABILITY.md`.
4. **Treinamento:** Conduzir sessão de lições aprendidas com a equipe.

## 6. Canais de Comunicação Internos

| **Canal** | **Uso** | **SLA** |
|---|---|---|
| Grupo de Incidentes (interno) | Coordenação em tempo real | Imediato |
| Email DPO | Comunicação formal ANPD/titulares | Até 72h |
| GitHub Issues (privado) | Rastreamento técnico do fix | Até 24h (contenção) |

## 7. Comunicação de Falhas de IA ao CRM/CFM (CFM 2.454/2026 Art. 7°, §2)

### 7.1 Obrigação Legal

O Art. 7°, §2 da Resolução CFM 2.454/2026 determina:

> *"É dever do médico comunicar às instâncias competentes eventuais falhas, riscos relevantes ou usos inadequados de modelos, sistemas e aplicações de IA que possam comprometer a segurança do paciente ou a qualidade da assistência."*

### 7.2 Escopo — Eventos Reportáveis ao CRM

Além dos incidentes de vazamento de dados (Seção 5), os seguintes eventos relacionados à IA devem ser comunicados ao CRM:

| **Evento** | **Exemplo** | **Critério de Reporte** |
|---|---|---|
| Falha do Safety Gate | `isCrisisInput` não detecta expressão suicida conhecida. | Sempre — qualquer falha do Safety Gate é reportável. |
| Recomendação clinicamente perigosa | LLM sugere interação medicamentosa contraindicada. | Quando a recomendação poderia causar dano se seguida. |
| Viés discriminatório crítico | IA fornece orientações qualitativamente inferiores para grupo protegido. | Quando classificado como Crítico ou Inaceitável (BIAS_MONITORING.md §5.1). |
| Indisponibilidade prolongada do Safety Gate | Failover não funciona; chat opera sem proteção de crise. | Qualquer indisponibilidade > 1 hora. |

### 7.3 Procedimento de Comunicação

| **Etapa** | **Ação** | **Responsável** | **Prazo** |
|---|---|---|---|
| 1 | Identificação e documentação da falha de IA. | Engenharia + RT | Imediato |
| 2 | Avaliação de impacto clínico pela Comissão de IA. | Comissão de IA (AI_GOVERNANCE.md) | 24 horas |
| 3 | Decisão sobre necessidade de comunicação ao CRM. | Coordenador Médico (RT) | 24 horas |
| 4 | Preparação de relatório formal ao CRM. | RT + DPO | 72 horas |
| 5 | Envio da comunicação ao CRM da jurisdição. | RT | 72 horas |
| 6 | Acompanhamento e resposta a solicitações do CRM. | RT + Gerência Executiva | Contínuo |

### 7.4 Conteúdo da Comunicação ao CRM

A comunicação formal deve conter:

1. Identificação do sistema de IA (Aura+, modelo Claude, versão snapshot).
2. Descrição da falha ou risco identificado.
3. Data e circunstância da ocorrência.
4. Avaliação do impacto clínico (real ou potencial).
5. Medidas corretivas adotadas ou em andamento.
6. Classificação de risco atualizada (RISK_CLASSIFICATION.md).

## 8. Testes e Simulações

- **Frequência:** Simulação de incidente ("tabletop exercise") a cada 6 meses.
- **Escopo:** Simular um cenário de severidade Crítica para validar tempos de resposta e fluxo de notificação.
- **Registro:** Resultados documentados e armazenados como evidência de compliance.

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
