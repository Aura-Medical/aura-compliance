# Aura Medical --- Dossiê de Compliance & Regulação

Repositório centralizado de documentação regulatória e compliance da plataforma Aura Medical.

Este dossiê cobre as exigências da **Anvisa (RDC 657/2022 e RDC 665/2022)**, **LGPD (Lei nº 13.709/2018)**, **CFM (Resolução 2.314/2022 sobre IA)** e as normas internacionais **IEC 62304** e **ISO 14971** para Software como Dispositivo Médico (SaMD) de Classe II.

🏗 Arquitetura do Sistema (v1.0.0)
----------------------------------

A arquitetura da Aura Medical foi simplificada e consolidada em dois repositórios principais, garantindo maior controle de versão e segurança clínica:

| **Repositório**                  | **Papel no Sistema**       | **Componentes Críticos de Compliance**                                                                                                      |
| --------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`aura-ios`** (Swift)          | Motor Clínico & App do Paciente | `DomainEvaluator.swift` (Lógica Binária Medicine 3.0), Instrumentos Clínicos (PHQ-9, PSQI), Segurança do Dispositivo (Keychain, App Attest). |
| **`aura-backend`** (TypeScript) | Orquestração Aura+, IA e Dados | `safety.ts` (Gate de Crise), `audit.ts` (Logs CFM 2.314), `gate.ts` (Doctor-in-the-loop), Integração Supabase (RLS).                       |

📂 Estrutura do Dossiê
-----------------------

Plaintext

```
samd/                  # Dossiê Técnico SaMD (Anvisa / IEC 62304 / ISO 14971)
  ├── INTENDED_USE.md  # Indicação de uso, exclusões de diagnóstico e regras do CDSS
  ├── RISK_ANALYSIS.md # Análise de Risco FMEA (Falsos negativos, Alucinação de IA, Vazamentos)
  ├── TRACEABILITY.md  # Matriz de Rastreabilidade (Requisitos → Código → Riscos)
  ├── VERIFICATION.md  # Plano de testes, validação clínica e metas de cobertura
  └── CONFIG_MGMT.md   # Controle de versão, releases e fluxo de CI/CD para o iOS e Backend

lgpd_cfm/              # Governança de Dados e Ética em IA (Substitui antiga pasta hipaa/)
  ├── BREACH_SOP.md    # Procedimento Operacional Padrão para vazamentos (Notificação 72h)
  ├── AI_AUDIT.md      # Regras de hashing SHA-256 para prompts de IA (CFM Art. 5)
  └── CONSENT.md       # Gestão de opt-in explícito para processamento de LLM (LGPD Art. 11)

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
