# Relatório de Gerenciamento de Riscos

**ID do Documento:** RR-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend (v1.0.0)

**Norma:** ISO 14971:2019 §9 — Relatório de Gerenciamento de Riscos

**Classificação:** SaMD Classe II (Anvisa RDC 657/2022), IMDRF Categoria II

---

## 1. Propósito

Este documento constitui o relatório final de gerenciamento de riscos conforme ISO 14971:2019 §9, sintetizando os resultados da análise FMEA documentada em `RISK_ANALYSIS.md` (RA-001 Rev 4.0) e declarando formalmente a aceitabilidade do risco residual global do produto.

## 2. Resumo do Processo de Gerenciamento de Riscos

### 2.1 Metodologia

- **Norma:** ISO 14971:2019
- **Método:** Análise de Modos de Falha e Efeitos (FMEA)
- **Fórmula:** RPN = Severidade (1–5) × Probabilidade (1–5) × Detecção (1–5)
- **Critérios de Aceitabilidade:** RPN ≤ 39 = Aceitável ou Baixo Risco; RPN 40–74 = Médio Risco (requer mitigação adicional); RPN ≥ 75 = Inaceitável.
- **Documento de Análise:** `RISK_ANALYSIS.md` (RA-001 Rev 4.0)

### 2.2 Escopo

A análise cobriu todos os perigos identificados para o sistema Aura Medical v1.0.0, incluindo:

- Avaliações binárias determinísticas (5 domínios de saúde)
- Interações com LLM (IA Generativa — Aura+)
- Instrumentos clínicos (PHQ-9, GAD-7, PSQI, FINDRISC)
- Integridade e sincronização de dados
- Segurança cibernética e proteção de PHI
- Conformidade com LGPD e CFM 2.454/2026

## 3. Resultados

### 3.1 Perigos Identificados

Foram identificados **17 perigos** (HAZ-01 a HAZ-17), todos com mitigações implementadas e verificadas.

### 3.2 Distribuição de Risco Residual

| Faixa de RPN | Classificação | Quantidade | IDs |
|---|---|---|---|
| 1–15 | Risco Aceitável | 9 | HAZ-04, HAZ-05, HAZ-06, HAZ-08, HAZ-13, HAZ-14, HAZ-15, HAZ-16, HAZ-17 |
| 16–39 | Risco Baixo | 8 | HAZ-01, HAZ-02, HAZ-03, HAZ-07, HAZ-09, HAZ-10, HAZ-11, HAZ-12 |
| 40–74 | Risco Médio | 0 | Nenhum |
| 75–125 | Risco Alto/Inaceitável | 0 | Nenhum |

**Nenhum perigo reside nas faixas "Médio" ou "Alto".**

### 3.3 Controles de Risco Críticos

Três controles foram designados como críticos para a manutenção da classificação SaMD Classe II (vide `RISK_ANALYSIS.md` §3.1):

| Controle | Implementação | Perigos Mitigados |
|---|---|---|
| Safety Gate | `safety.ts:isCrisisInput()` — 33 padrões regex PT-BR, bypass pré-LLM | HAZ-02, HAZ-03 |
| Doctor-in-the-Loop | `gate.ts:checkAuraPlusGate()` — protocolos bloqueados sem validação médica | HAZ-02, HAZ-04 |
| Trilha de Auditoria | `audit.ts:logAiInteraction()` — SHA-256, fail-soft, modelo versionado | HAZ-05, HAZ-12 |

### 3.4 Perigos de Maior Severidade

| Perigo | Severidade | RPN Final | Mitigação Principal |
|---|---|---|---|
| HAZ-01 (Falso Saudável) | 5 (Catastrófico) | 30 | Ghost Mode + Data Caps + Indicador de Precisão |
| HAZ-02 (Alucinação de IA) | 5 (Catastrófico) | 30 | System prompt guardrails + Catálogo estático + Disclaimer |
| HAZ-03 (Falha na Detecção de Crise) | 5 (Catastrófico) | 20 | Safety Gate (33 regex) + PHQ-9 safety flag (90 dias) |
| HAZ-05 (PHI sem Consentimento) | 5 (Catastrófico) | 10 | Consent gate fail-closed + Hashing SHA-256 |
| HAZ-08 (Violação de BD) | 5 (Catastrófico) | 10 | RLS + Criptografia em repouso + Chave de serviço isolada |

## 4. Análise Benefício-Risco (ISO 14971 §7)

### 4.1 Benefícios Clínicos

- Detecção precoce de desregulação metabólica e cardiovascular em indivíduos assintomáticos.
- Interceptação de crises agudas (ideação suicida) com escalonamento imediato para emergência humana.
- Educação em saúde preventiva personalizada baseada em evidências (Medicine 3.0).
- Monitoramento contínuo de 5 domínios de carga alostática integrando dados laboratoriais, wearables e instrumentos clínicos validados.

### 4.2 Riscos Residuais

- Todos os 17 perigos possuem RPN ≤ 30.
- Os 3 controles críticos fornecem defesa em profundidade contra os perigos de maior severidade.
- O Doctor-in-the-Loop impede autonomia clínica por design.
- O sistema opera exclusivamente em contextos de saúde não-críticos (prevenção e longevidade).

### 4.3 Conclusão Benefício-Risco

Os benefícios clínicos identificados superam significativamente os riscos residuais. O perfil de risco é consistente com SaMD Classe II (RDC 657/2022 Art. 7°, II) e IMDRF Categoria II.

## 5. Conformidade com Requisitos de Risco para IA (CFM 2.454/2026)

A Resolução CFM 2.454/2026 introduz requisitos específicos de risco para IA na medicina:

| Requisito CFM | Implementação | Documento |
|---|---|---|
| Arts. 12–13 (Classificação de risco) | IA classificada como médio risco | `lgpd_cfm/RISK_CLASSIFICATION.md` |
| Art. 14 (Comissão de IA) | Comissão instituída sob coordenação do RT | `lgpd_cfm/AI_GOVERNANCE.md` |
| Anexo III, II (Viés) | Monitoramento trimestral de viés | `lgpd_cfm/BIAS_MONITORING.md` |
| Anexo III, VI (Ciclo de vida) | Gestão do ciclo de vida da IA | `lgpd_cfm/LIFECYCLE_MGMT.md` |

## 6. Declaração de Aceitabilidade do Risco Residual

Com base na análise FMEA de 17 perigos, nas mitigações implementadas e verificadas, na análise benefício-risco e na conformidade com ISO 14971:2019, RDC 657/2022, RDC 665/2022 e CFM 2.454/2026:

**O risco residual global do produto Aura Medical v1.0.0 é declarado ACEITÁVEL.**

Esta declaração está condicionada à manutenção dos 3 controles de risco críticos (Safety Gate, Doctor-in-the-Loop, Trilha de Auditoria) e ao monitoramento pós-mercado conforme `lgpd_cfm/BIAS_MONITORING.md` e `lgpd_cfm/AI_GOVERNANCE.md`.

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Gestor de Risco (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gestor de Qualidade | Frederico | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
