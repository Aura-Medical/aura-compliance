# Registro Mestre de Produto (RMP)

**ID do Documento:** RMP-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend (v1.0.0)

**Fabricante:** Auramedical Tecnologia Ltda.

**Responsável Técnico:** Dr. Alexandre Teixeira de Almeida

**Classificação:** SaMD Classe II (Anvisa RDC 657/2022), IMDRF Categoria II, IEC 62304 Classe B

**Norma:** RDC 665/2022 Capítulo VI — Rastreabilidade e Registro Mestre de Produto

---

## 1. Propósito

Este documento constitui o Registro Mestre de Produto (RMP) da Aura Medical, conforme exigido pela RDC 665/2022. O RMP é o índice mestre que identifica e referencia todos os documentos, registros e artefatos que compõem o dossiê técnico do produto.

## 2. Identificação do Produto

| Campo | Informação |
|---|---|
| Nome comercial | Aura Medical |
| Módulo de IA | Aura+ |
| Versão | 1.0.0 |
| Plataforma | iOS 17+ |
| Classificação Anvisa | SaMD Classe II (RDC 657/2022 Art. 7°, II) |
| Classificação IMDRF | Categoria II (N12) |
| Classificação IEC 62304 | Classe B |
| Classificação de IA (CFM) | Médio Risco (Resolução 2.454/2026) |
| Bundle ID | med.aura.medical |
| Team ID | RZ9WFC8ZMG |

## 3. Índice do Dossiê Técnico

### 3.1 Documentos SaMD (Dossiê Técnico Principal)

| # | Documento | ID | Rev | Arquivo | Status |
|---|---|---|---|---|---|
| 1 | Plano de Desenvolvimento de Software | SDP-001 | 1.0 | `samd/SDP.md` | Vigente |
| 2 | Declaração de Indicação de Uso | IU-001 | 4.0 | `samd/INTENDED_USE.md` | Vigente |
| 3 | Análise de Risco (FMEA) | RA-001 | 4.0 | `samd/RISK_ANALYSIS.md` | Vigente |
| 4 | Relatório de Gerenciamento de Riscos | RR-001 | 1.0 | `samd/RISK_REPORT.md` | Vigente |
| 5 | Matriz de Rastreabilidade | TM-001 | 4.0 | `samd/TRACEABILITY.md` | Vigente |
| 6 | Plano de Verificação e Validação | VV-001 | 4.0 | `samd/VERIFICATION.md` | Vigente |
| 7 | Relatório de V&V (Execução) | VV-002 | — | `samd/VV_REPORT.md` | **Pendente** (requer execução de testes) |
| 8 | Gerenciamento de Configuração | CM-001 | 4.0 | `samd/CONFIG_MGMT.md` | Vigente |
| 9 | Software Bill of Materials | SB-001 | 1.0 | `samd/SBOM.md` | Vigente |
| 10 | Instruções de Uso | IF-001 | 1.0 | `samd/IFU.md` | Vigente |
| 11 | Declaração de Conformidade | DC-001 | 1.0 | `samd/DECLARATION.md` | Vigente (pendente assinatura) |
| 12 | Registro Mestre de Produto | RMP-001 | 1.0 | `samd/RMP.md` | **Este documento** |

### 3.2 Governança de Dados e IA (CFM 2.454/2026 + LGPD)

| # | Documento | ID | Rev | Arquivo | Status |
|---|---|---|---|---|---|
| 13 | Política de Auditoria de IA | AA-001 | 1.0 | `lgpd_cfm/AI_AUDIT.md` | Vigente |
| 14 | Comissão de IA e Telemedicina | GV-001 | 1.0 | `lgpd_cfm/AI_GOVERNANCE.md` | Vigente |
| 15 | Monitoramento de Viés | BM-001 | 1.0 | `lgpd_cfm/BIAS_MONITORING.md` | Vigente |
| 16 | SOP de Resposta a Incidentes | BR-001 | 1.0 | `lgpd_cfm/BREACH_SOP.md` | Vigente |
| 17 | Gestão de Consentimento | CN-001 | 1.0 | `lgpd_cfm/CONSENT.md` | Vigente |
| 18 | Ciclo de Vida da IA | LC-001 | 1.0 | `lgpd_cfm/LIFECYCLE_MGMT.md` | Vigente |
| 19 | Classificação de Risco de IA | RC-001 | 1.0 | `lgpd_cfm/RISK_CLASSIFICATION.md` | Vigente |

### 3.3 Artefatos de Software

| # | Artefato | Localização | Controle |
|---|---|---|---|
| 20 | Código-Fonte iOS | `aura-ios/` (GitHub) | Git + SemVer |
| 21 | Código-Fonte Backend | `aura-backend/` (GitHub) | Git + SemVer |
| 22 | Tags de Release | Git tags (ex: `v1.0.0`) | Imutáveis |
| 23 | Logs de Auditoria de IA | Tabela `ai_audit_log` (Supabase) | RLS + Retenção 5 anos |
| 24 | Registros de Consentimento | Tabela `user_consents` (Supabase) | RLS + Retenção 5 anos |

## 4. Itens Pendentes

| Item | Dependência | Responsável |
|---|---|---|
| VV_REPORT.md (Relatório de V&V) | Execução real dos 24 test cases | Engenharia + RT |
| Assinaturas em todos os documentos | Coleta de assinaturas (digital ou ICP-Brasil) | Todos |
| DPA com Anthropic | Negociação contratual | Gerência Executiva + DPO |

## 5. Controle de Revisões do RMP

| Revisão | Data | Descrição | Autor |
|---|---|---|---|
| 1.0 | 2026-04-10 | Criação inicial — dossiê completo v1.0.0 | Dr. Alexandre T. de Almeida |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Compliance e Qualidade | Frederico | | |
