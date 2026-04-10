# Gestão do Ciclo de Vida — Inteligência Artificial como Produto

**ID do Documento:** LC-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical — Módulo Aura+ (IA Conversacional)

**Norma:** Resolução CFM 2.454/2026 Anexo III, item VI; Anexo I, item IV

---

## 1. Propósito

Este documento define o processo de gestão do ciclo de vida do módulo Aura+ como produto de IA, conforme exigido pelo Anexo III, item VI, da Resolução CFM 2.454/2026, que determina o tratamento de soluções de IA com práticas de gestão de produto incluindo fases definidas de requisitos, desenvolvimento, validação, implantação, suporte e melhorias contínuas.

## 2. Definição de Ciclo de Vida (Anexo I, IV)

> *"A série de fases que compreende a concepção, o planejamento, o desenvolvimento, o treinamento, o retreinamento, os testes, a validação, a implantação, o monitoramento, as eventuais modificações ou adaptações de um sistema de IA incluindo sua descontinuação quando for o caso, bem como o acompanhamento de seus impactos após a implantação."*

## 3. Fases do Ciclo de Vida — Aura+

### 3.1 Fase 1 — Concepção e Requisitos

| **Atividade** | **Responsável** | **Artefato** |
|---|---|---|
| Levantamento de requisitos clínicos e funcionais | RT (Dr. Alexandre) | TRACEABILITY.md (REQs) |
| Avaliação preliminar de risco | Comissão de IA | RISK_CLASSIFICATION.md |
| Definição de finalidade e escopo da IA | Comissão de IA | Este documento |
| Avaliação de compatibilidade ética (Art. 9°, §1) | Comissão de IA | Ata de deliberação |

### 3.2 Fase 2 — Desenvolvimento

| **Atividade** | **Responsável** | **Artefato** |
|---|---|---|
| Implementação de funcionalidades | Engenharia | Código-fonte (Git) |
| Configuração do LLM (system prompt, guardrails) | Engenharia + RT | `system-prompt.ts`, `safety.ts` |
| Integração do Safety Gate | Engenharia | RISK_ANALYSIS.md (HAZ-01 a HAZ-04) |
| Implementação de controles de privacidade | Engenharia + DPO | CONSENT.md, AI_AUDIT.md |

### 3.3 Fase 3 — Testes e Validação

| **Atividade** | **Responsável** | **Artefato** |
|---|---|---|
| Testes funcionais (suíte automatizada) | Engenharia | VERIFICATION.md |
| Testes de segurança do paciente (Safety Gate) | Engenharia + RT | VERIFICATION.md §4.4 |
| Testes de consentimento (consent gate) | Engenharia | VERIFICATION.md (AI-04) |
| Teste de viés com prompts sintéticos | Comissão de IA | BIAS_MONITORING.md §4.3 |
| Validação clínica pelo RT | RT | Relatório de validação |

### 3.4 Fase 4 — Implantação

| **Atividade** | **Responsável** | **Artefato** |
|---|---|---|
| Aprovação da Comissão de IA para release | Comissão de IA | Ata de deliberação |
| Deploy em produção | Engenharia | CONFIG_MGMT.md (Change Control) |
| Comunicação ao utilizador (se mudança relevante) | DPO | CONSENT.md §6 (re-consentimento) |
| Atualização de documentação regulatória | RT | Dossiê SaMD |

### 3.5 Fase 5 — Monitoramento Pós-Implantação

| **Atividade** | **Responsável** | **Frequência** | **Artefato** |
|---|---|---|---|
| Auditoria de logs de IA | Engenharia | Mensal | AI_AUDIT.md §5.1 |
| Monitoramento de viés | Comissão de IA | Trimestral | BIAS_MONITORING.md |
| Revisão de classificação de risco | Comissão de IA | Anual ou sob demanda | RISK_CLASSIFICATION.md §7 |
| Acompanhamento de incidentes | Líder de Incidente | Contínuo | BREACH_SOP.md |
| Verificação de versão do modelo LLM | Engenharia | Contínuo | AI_AUDIT.md §4.4 |

### 3.6 Fase 6 — Atualizações e Retreinamento

| **Atividade** | **Responsável** | **Controle** |
|---|---|---|
| Atualização de versão do LLM | Engenharia | Aprovação do RT + Comissão. Registro em CONFIG_MGMT.md. |
| Modificação do system prompt | Engenharia + RT | Change Control (CONFIG_MGMT.md §4). Tag Git obrigatória. |
| Adição de nova funcionalidade de IA | Engenharia | Retorno à Fase 1 (requisitos → risco → desenvolvimento → validação). |
| Correção de viés detectado | Engenharia + RT | BIAS_MONITORING.md §5.2. Ciclo expedito: correção → teste → deploy. |

### 3.7 Fase 7 — Descontinuação (se aplicável)

| **Atividade** | **Responsável** | **Controle** |
|---|---|---|
| Deliberação de descontinuação | Comissão de IA | Quórum com Coordenador Médico. |
| Comunicação aos utilizadores | DPO | Notificação com prazo mínimo de 30 dias. |
| Preservação de registros de auditoria | Engenharia | Retenção conforme AI_AUDIT.md §6 (5 anos). |
| Documentação de lições aprendidas | Comissão de IA | Relatório final. |

## 4. Controle de Mudanças

Toda mudança no módulo Aura+ é classificada conforme CONFIG_MGMT.md:

| **Categoria** | **Exemplo** | **Aprovação** | **Retorno ao Ciclo** |
|---|---|---|---|
| **Rotina** | Correção de bug sem impacto na IA | Engenharia | Fase 3 (testes) → Fase 4 |
| **Significativa** | Nova funcionalidade de IA, mudança de modelo | RT + Comissão | Fase 1 (requisitos) → todas |
| **Emergência** | Safety Gate comprometido, viés crítico | RT (imediato), Comissão (ratificação) | Fase 2 → Fase 3 → Fase 4 |

## 5. Compatibilidade Ética em Todo o Ciclo (Art. 9°, §1)

Conforme Art. 9°, §1, a verificação de compatibilidade com direitos fundamentais, ética médica e bioética ocorre em **todas as fases**:

| **Fase** | **Verificação Ética** |
|---|---|
| Concepção | Avaliação de risco + deliberação da Comissão. |
| Desenvolvimento | Privacy by design (Anexo I, XV). Minimização de dados (LGPD Art. 6). |
| Testes | Testes de viés. Validação do Safety Gate. |
| Implantação | Consentimento informado. Comunicação ao paciente. |
| Monitoramento | Monitoramento de viés. Auditoria contínua. |
| Atualização | Reavaliação de risco. Re-consentimento se necessário. |
| Descontinuação | Preservação de direitos do titular. Retenção de evidências. |

## 6. Rastreabilidade

| **Requisito** | **Vínculo** |
|---|---|
| Gestão do ciclo de vida (Anexo III, VI) | Este documento |
| Definição de ciclo de vida (Anexo I, IV) | Seção 2 |
| Compatibilidade ética em todas as fases (Art. 9°, §1) | Seção 5 |
| Controle de mudanças | CONFIG_MGMT.md |
| Auditoria e monitoramento (Art. 9°, §2) | AI_AUDIT.md, BIAS_MONITORING.md |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
