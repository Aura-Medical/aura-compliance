# Aura Medical --- Dossiê de Compliance & Regulação

Repositório centralizado de documentação regulatória e compliance da plataforma Aura Medical.

Este dossiê cobre as exigências da **Anvisa (RDC 657/2022 e RDC 665/2022)**, **LGPD (Lei nº 13.709/2018)**, **CFM (Resolução 2.454/2026 sobre IA)** e as normas internacionais **IEC 62304** e **ISO 14971** para Software como Dispositivo Médico (SaMD) de Classe II.

🏗 Arquitetura do Sistema (v1.0.0)
----------------------------------

A arquitetura da Aura Medical foi simplificada e consolidada em dois repositórios principais, garantindo maior controle de versão e segurança clínica:

| **Repositório**                  | **Papel no Sistema**       | **Componentes Críticos de Compliance**                                                                                                      |
| --------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`aura-ios`** (Swift)          | Motor Clínico & App do Paciente | `DomainEvaluator.swift` (Lógica Binária Medicine 3.0), Instrumentos Clínicos (PHQ-9, PSQI), Segurança do Dispositivo (Keychain, App Attest). |
| **`aura-backend`** (TypeScript) | Orquestração Aura+, IA e Dados | `safety.ts` (Gate de Crise), `audit.ts` (Logs CFM 2.454/2026), `gate.ts` (Doctor-in-the-loop), Integração Supabase (RLS).                       |

📂 Estrutura do Dossiê
-----------------------

Plaintext

```
samd/                    # Dossiê Técnico SaMD (Anvisa / IEC 62304 / ISO 14971)
  ├── RMP.md             # Registro Mestre de Produto — índice mestre do dossiê
  ├── SDP.md             # Plano de Desenvolvimento de Software (documento-raiz)
  ├── INTENDED_USE.md    # Indicação de uso, classificação CDSS e regras do sistema
  ├── RISK_ANALYSIS.md   # Análise de Risco FMEA (17 HAZ, 3 controles críticos)
  ├── RISK_REPORT.md     # Relatório de Gerenciamento de Riscos (ISO 14971 §9)
  ├── TRACEABILITY.md    # Matriz de Rastreabilidade (33 REQ → Código → Teste → HAZ)
  ├── VERIFICATION.md    # Plano de V&V (24 test cases, metas de cobertura)
  ├── CONFIG_MGMT.md     # Controle de versão, releases e BPF digital
  ├── SBOM.md            # Software Bill of Materials (dependências de terceiros)
  ├── IFU.md             # Instruções de Uso ao utilizador (RDC 751/2022)
  └── DECLARATION.md     # Declaração de Conformidade do Fabricante

lgpd_cfm/                    # Governança de Dados e Ética em IA (CFM 2.454/2026 + LGPD)
  ├── AI_AUDIT.md            # Regras de hashing SHA-256 para prompts de IA (CFM Art. 9°)
  ├── AI_GOVERNANCE.md       # Comissão de IA e Telemedicina (CFM Art. 14)
  ├── BIAS_MONITORING.md     # Monitoramento de viés discriminatório (CFM Anexo III)
  ├── BREACH_SOP.md          # SOP de vazamentos e falhas de IA (Notificação 72h)
  ├── CONSENT.md             # Gestão de opt-in para processamento por LLM (LGPD Art. 11)
  ├── LIFECYCLE_MGMT.md      # Ciclo de vida da IA como produto (CFM Anexo III)
  └── RISK_CLASSIFICATION.md # Classificação de risco de IA (CFM Arts. 12-13)

```

🏷 Classificação Regulatória (SaMD)
--------------------------------------

- **Classe de Risco:** Classe II (Anvisa RDC 657/2022) / IMDRF Categoria II.
- **Propósito (Intended Use):** Suporte à Decisão Clínica (CDSS) para otimização de longevidade e triagem de saúde preventiva, utilizando o Modelo de Seeman (limiares binários) e IA conversacional parametrizada.
- **O que o software NÃO É:** * Não é diagnóstico.

  - Não é prescritivo (medicamentos/marcas).
  - Não é autônomo (Protocolos de intervenção são estritamente bloqueados até a validação presencial ou por telemedicina de um médico --- *Doctor-in-the-loop*).

🔒 Postura de Segurança e Privacidade
--------------------------------------

A Aura Medical adota uma postura de **Security e Privacy by Design**:

- **Proteção de PHI:** Dados de saúde protegidos por *Row-Level Security* (RLS) no banco de dados, criptografia em repouso (SwiftData) e ofuscação de tela no iOS.
- **IA Segura:** A inteligência clínica (Aura+) possui um interceptador determinístico (Safety Gate) que impede o uso do LLM em situações de crise aguda (ex: ideação suicida), acionando fluxos de emergência imediatamente.
- **Transparência:** Todos os outputs do sistema incluem um indicador de "Confiança de Dados" (Precision Indicator) e disclaimers obrigatórios.
