# Plano de Desenvolvimento de Software (SDP)

**ID do Documento:** SDP-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend (v1.0.0)

**Fabricante:** Auramedical Tecnologia Ltda.

**Normas:** IEC 62304:2006+AMD1:2015; RDC 665/2022 (BPF); RDC 657/2022 (SaMD); ISO 14971:2019

**Classificação:** SaMD Classe II (Anvisa), IMDRF Categoria II, IEC 62304 Classe B

---

## 1. Propósito

Este documento é o plano-raiz do ciclo de desenvolvimento do software Aura Medical, conforme exigido pela IEC 62304:2006+AMD1:2015 §5.1.1. Define o modelo de ciclo de vida, processos, entregas, papéis, ferramentas e critérios de aceitação que governam todo o desenvolvimento do SaMD.

Todos os demais documentos do Sistema de Gestão da Qualidade (SGQ) são subordinados a este plano.

## 2. Escopo do Produto

### 2.1 Descrição

A Aura Medical é um sistema de suporte à decisão clínica (CDSS) para iOS, composto por:

- **Motor Clínico Determinístico (iOS):** Avaliador binário de 5 domínios de saúde baseado no Modelo de Seeman, com instrumentos clínicos validados (PHQ-9, GAD-7, PSQI, FINDRISC).
- **Motor de IA Conversacional (Backend):** Módulo Aura+ com LLM (Claude Haiku) parametrizado por safety gates, guardrails editoriais e protocolos baseados em evidências.

### 2.2 Classificação Regulatória

| Regulador | Classificação | Referência |
|---|---|---|
| Anvisa | SaMD Classe II (Médio Risco) | RDC 657/2022 Art. 7°, II |
| IMDRF | Categoria II | N12 — "Conduzir Gestão Clínica" × "Não-Crítico" |
| IEC 62304 | Classe B | Falha pode contribuir para lesão, mas não morte |
| CFM | Médio Risco (IA) | Resolução 2.454/2026 Arts. 12–13 |

A justificativa completa está em `INTENDED_USE.md` (IU-001) §5.

### 2.3 Finalidade Pretendida (Resumo)

Suporte à decisão clínica em saúde preventiva e longevidade (Medicine 3.0) para adultos saudáveis. O software **não diagnostica**, **não prescreve** e **não executa intervenções sem validação médica** (Doctor-in-the-Loop). Vide `INTENDED_USE.md` para a declaração completa.

## 3. Modelo de Ciclo de Vida

O desenvolvimento adota um modelo **iterativo incremental** organizado em Ondas:

| Onda | Escopo | Status |
|---|---|---|
| **Onda 1** | Motor clínico binário + Aura+ com guardrails + Safety Gate + Audit Trail. Protocolos locked (preview only). | v1.0.0 — Entregue |
| **Onda 2** | UI de consentimento LGPD + Webhook de desbloqueio médico + Telemedicina. | Planejada |
| **Onda 3** | RAG sobre corpus clínico + Protocolos dinâmicos + Expansão de domínios. | Planejada |

Cada Onda segue o ciclo: Requisitos → Design → Implementação → Verificação → Validação Clínica → Release.

Para o ciclo de vida específico do módulo de IA, vide `lgpd_cfm/LIFECYCLE_MGMT.md` (LC-001).

## 4. Processos de Desenvolvimento

### 4.1 Gerenciamento de Requisitos

- **Documento:** `TRACEABILITY.md` (TM-001)
- **Processo:** Cada requisito (REQ-XX) é vinculado a design, código-fonte, teste e perigo (HAZ-XX), formando a cadeia de rastreabilidade férrea: REQ → Código → Teste → HAZ.
- **Aprovação:** Novos requisitos clínicos exigem aprovação do RT.

### 4.2 Gerenciamento de Riscos

- **Documento:** `RISK_ANALYSIS.md` (RA-001)
- **Norma:** ISO 14971:2019
- **Método:** FMEA com RPN = Severidade × Probabilidade × Detecção
- **Processo:** Perigos identificados durante design e implementação são registrados como HAZ-XX, mitigados via REQ-XX, e verificados via test cases.
- **Relatório Final:** `RISK_REPORT.md` (RR-001) — lacre do processo de risco.

### 4.3 Gerenciamento de Configuração

- **Documento:** `CONFIG_MGMT.md` (CM-001)
- **Ferramentas:** Git/GitHub (controle de versão), SemVer (versionamento), xcodegen (build iOS).
- **Independência de Revisão:** Mudanças em código crítico exigem revisor ≠ autor (RDC 665/2022).
- **SBOM:** `SBOM.md` — inventário de dependências de terceiros.

### 4.4 Verificação e Validação

- **Plano:** `VERIFICATION.md` (VV-001) — 24 test cases cobrindo 33 requisitos.
- **Relatório:** `VV_REPORT.md` — evidência de execução dos testes com resultados reais.
- **Metas de Cobertura:** DomainEvaluator.swift >90%, safety.ts >95%, Segurança >80%.
- **Validação Clínica:** Revisão médica pelo RT contra guidelines (ADA 2024, AHA PREVENT).

### 4.5 Proteção de Dados e Governança de IA

- **Documentos:** 7 documentos em `lgpd_cfm/` cobrindo LGPD, CFM 2.454/2026 e governança de IA.
- **Comissão de IA:** `AI_GOVERNANCE.md` (GV-001) — comissão sob coordenação médica (CFM Art. 14).

## 5. Papéis e Responsabilidades

| Papel | Responsável | Atribuições no SDP |
|---|---|---|
| **Responsável Técnico (RT)** | Dr. Alexandre Teixeira de Almeida | Define requisitos clínicos, valida limiares, aprova releases com impacto clínico, coordena a Comissão de IA. |
| **Gerência Executiva / Engenharia** | Arthur Teixeira de Almeida | Implementação técnica, controle de configuração, processo de build e release, contenção de incidentes. |
| **Compliance e Qualidade (SGQ)** | Frederico | Garantidor do SGQ, revisão de documentação regulatória, interface com ANPD, independência de revisão. |

### 5.1 Independência de Revisão (RDC 665/2022)

Detalhada em `VERIFICATION.md` (VV-001) §1.2. Princípio: nenhum artefato "Crítico" pode ser aprovado exclusivamente por seu autor.

## 6. Ferramentas de Desenvolvimento

Detalhadas em `CONFIG_MGMT.md` (CM-001) §7. Resumo:

| Ferramenta | Propósito |
|---|---|
| Xcode 16.x+ / Swift | IDE e compilação iOS |
| xcodegen | Geração determinística de projeto |
| Node.js LTS (v20+) / TypeScript | Runtime do backend |
| Hono | Framework HTTP do backend |
| Git / GitHub | Controle de versão e PRs |
| Supabase (PostgreSQL 15+) | BaaS, Auth, RLS, Logs |
| Vitest | Framework de testes backend |
| XCTest | Framework de testes iOS |
| TestFlight | Distribuição de QA |

## 7. Entregas do Projeto

| Entrega | Documento | Tipo |
|---|---|---|
| Indicação de Uso | `INTENDED_USE.md` (IU-001) | Regulatório |
| Análise de Risco (FMEA) | `RISK_ANALYSIS.md` (RA-001) | Regulatório |
| Relatório de Risco | `RISK_REPORT.md` (RR-001) | Regulatório |
| Matriz de Rastreabilidade | `TRACEABILITY.md` (TM-001) | Regulatório |
| Plano de V&V | `VERIFICATION.md` (VV-001) | Regulatório |
| Relatório de V&V | `VV_REPORT.md` (pendente execução) | Regulatório |
| Gerenciamento de Configuração | `CONFIG_MGMT.md` (CM-001) | Regulatório |
| SBOM | `SBOM.md` | Regulatório |
| Instruções de Uso | `IFU.md` | Regulatório |
| Declaração de Conformidade | `DECLARATION.md` | Regulatório |
| Governança de Dados e IA | `lgpd_cfm/` (7 documentos) | Regulatório |
| Registro Mestre de Produto | `RMP.md` | Índice Mestre |
| Código-Fonte | `aura-ios/`, `aura-backend/` | Artefato |
| Tags de Release | Git tags (ex: `v1.0.0`) | Artefato |

## 8. Critérios de Aceitação de Release

Definidos em `VERIFICATION.md` §8:

1. Zero falhas nas suítes de avaliação binária e instrumentos.
2. Zero falhas na suíte de segurança de IA (Safety Gate).
3. Cobertura de código do `DomainEvaluator.swift` ≥ 90%.
4. Assinatura formal do RT para novos biomarcadores ou protocolos.
5. Nenhum defeito Crítico ou Alto em aberto.
6. Aprovação da Comissão de IA para mudanças que afetam o módulo Aura+.

## 9. Referências Cruzadas

| Documento | Relação |
|---|---|
| `INTENDED_USE.md` (IU-001) | Finalidade pretendida e classificação |
| `RISK_ANALYSIS.md` (RA-001) | Processo de gerenciamento de riscos |
| `RISK_REPORT.md` (RR-001) | Relatório final de riscos |
| `TRACEABILITY.md` (TM-001) | Rastreabilidade de requisitos |
| `VERIFICATION.md` (VV-001) | Plano de verificação e validação |
| `CONFIG_MGMT.md` (CM-001) | Gerenciamento de configuração |
| `SBOM.md` | Inventário de dependências |
| `IFU.md` | Instruções de uso |
| `DECLARATION.md` | Declaração de conformidade |
| `lgpd_cfm/` | Governança de dados e IA (7 docs) |
| `RMP.md` | Registro Mestre de Produto (índice) |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Compliance e Qualidade | Frederico | | |
