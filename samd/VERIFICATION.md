# Plano de Verificação e Validação

**ID do Documento:** VV-001

**Revisão:** 4.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend

**Norma:** IEC 62304:2006+AMD1:2015 Seção 5.7 (Verificação de Software)

---

## 1. Escopo

Este documento descreve a estratégia de verificação e validação para o sistema Aura Medical (App iOS e Backend Aura+), cobrindo testes unitários, testes do motor clínico binário, testes de segurança e guardrails de Inteligência Artificial. O plano garante que todos os 33 requisitos definidos na Matriz de Rastreabilidade (`TRACEABILITY.md`, TM-001 Rev 3.0) sejam rigorosamente verificados.

### 1.1 Base Regulatória

Este plano de verificação atende aos seguintes requisitos regulatórios:

- **IEC 62304:2006+AMD1:2015 §5.7** — Verificação de software médico.
- **RDC 665/2022 (Boas Práticas de Fabricação para Dispositivos Médicos):**
  - **Capítulo IV, Seção III (Verificação e Validação de Projeto):** Exige verificação independente de que o design atende aos requisitos especificados.
  - **Princípio de Independência de Revisão:** A verificação deve ser conduzida por pessoa(s) independente(s) daquela(s) que realizou(aram) o design.
- **IMDRF/SaMD WG/N23FINAL:2015 §5.4** — Verificação e validação de SaMD, incluindo testes funcionais, de desempenho e de segurança proporcionais à categoria de risco (Categoria II).

### 1.2 Independência de Revisão (RDC 665/2022)

Conforme RDC 665/2022 e ISO 14971:2019, as seguintes atribuições de independência são definidas para o SGQ da Aura Medical:

| **Artefato** | **Autor / Responsável** | **Revisor Independente** | **Racional** |
|---|---|---|---|
| Regras clínicas (`DomainEvaluator`, `InstrumentScoring`) | Dr. Alexandre (RT) | Arthur / Frederico | RT define limiares; engenharia/qualidade revisam implementação. |
| Safety Gate (`safety.ts`) | Engenharia | Dr. Alexandre (RT) | Engenharia implementa; RT valida adequação clínica dos padrões de crise. |
| System Prompt (`system-prompt.ts`) | Dr. Alexandre (RT) + Engenharia | Frederico (SGQ) | Conteúdo clínico revisado por garantidor de qualidade. |
| Trilha de Auditoria (`audit.ts`) | Engenharia | Frederico (SGQ) | QA garante conformidade com CFM 2.454/2026 Art. 9°. |
| Documentação Regulatória (`samd/`, `lgpd_cfm/`) | Dr. Alexandre (RT) | Frederico (SGQ) | RT é autor; SGQ é garantidor da qualidade documental. |

**Princípio:** Nenhum artefato classificado como "Crítico" (`CONFIG_MGMT.md` §4.1) pode ser aprovado exclusivamente por seu autor. A aprovação requer pelo menos um revisor independente.

**Garantidor do SGQ:** Frederico atua como garantidor do Sistema de Gestão da Qualidade (SGQ), assegurando que todos os processos de verificação e validação cumpram os requisitos regulatórios aplicáveis.

## 2. Visão Geral da Estratégia de Testes

| **Nível de Teste** | **Escopo** | **Framework** | **Automação** |
|---|---|---|---|
| **Testes Unitários Clínicos** | Limiares de biomarcadores, status de domínio e validação de inputs. | XCTest | CI |
| **Testes de Instrumentos** | Cálculos de PHQ-9, GAD-7, PSQI, FINDRISC e flags derivadas. | XCTest | CI |
| **Testes de Backend (Aura+)** | Detecção de crise (regex), System Prompt Context e Audit Logging. | Vitest | CI |
| **Testes de Segurança** | Auth Biométrica, Blur de PHI, Keychain, App Attest, RLS, Certificate Pinning. | XCTest + Vitest | Híbrido |
| **Validação Clínica** | Adequação das recomendações (Medicine 3.0) e curadoria do catálogo de protocolos. | Revisão Médica | Manual |

## 3. Metas de Cobertura

| **Componente** | **Meta de Cobertura** | **Racional (Conformidade SaMD)** |
|---|---|---|
| `DomainEvaluator.swift` | **>90%** | Exige altíssima confiança nas regras de flag clínico e limiares binários. |
| `aura-plus/safety.ts` | **>95%** | Módulo de contenção de risco de vida (detecção de crise) não pode apresentar falsos negativos. |
| Controles de Segurança | **>80%** | Defesa de PHI e cumprimento da LGPD Art. 11 exigem verificações exaustivas. |
| Geral do Sistema | **>60%** | Padrão mínimo para software de Classe II (Anvisa/IEC 62304). |

## 4. Suítes de Teste

### 4.1 Testes do Avaliador de Domínios

**Propósito:** Garantir que o `DomainEvaluator` conte corretamente as flags de risco com base nas diretrizes clínicas, aplicando Data Caps e respeitando o limite mínimo de itens (Ghost Mode).

**Vetores de Teste:**

| **ID** | **Cenário** | **Resultado Esperado** | **Validação Central** |
|---|---|---|---|
| **EV-01** | Homem saudável, dados completos | Todos os 5 domínios retornam status "Saudável" (Level = 0) | Caminho feliz total |
| **EV-02** | Dados insuficientes (< 3 itens) | Domínio retorna `level: nil` | Ghost Mode (REQ-13) |
| **EV-03** | ApoB = 92 mg/dL | Flag `apob_elevated` ativada | Data Cap de Alto Risco (REQ-12) |
| **EV-04** | Mulher com Ácido Úrico = 6.2 | Flag `uric_acid_elevated` ativada | Limiares Específicos por Sexo (REQ-24) |
| **EV-05** | Questionário Saudável + HbA1c 5.8% | Flag `hba1c_prediabetic` ativada | Lab sobrepõe Questionário (REQ-12) |
| **EV-06** | Dados de Garmin + Oura + Apple Watch | Domínios Sono e Movimento recebem inputs de 3 fontes | Ingestão Multi-Fonte (REQ-28) |

**Critérios de Aceite:**

- Exatamente 5 objetos `DomainStatus` retornados.
- Limiares aplicados condicionalmente à idade/sexo quando apropriado.
- *Requisitos Verificados:* REQ-01, REQ-02, REQ-12, REQ-13, REQ-15, REQ-24, REQ-25, REQ-28, REQ-34.

### 4.2 Testes de Instrumentos Clínicos Validados

**Propósito:** Validar que as escalas psiquiátricas e metabólicas produzem scores totais exatos e emitem as `derivedFlags` corretas, substituindo as flags básicas do onboarding.

**Casos de Teste:**

| **ID** | **Instrumento** | **Inputs** | **Resultado Esperado** |
|---|---|---|---|
| **INST-01** | PHQ-9 | Item 9 = 2, Total = 11 | `severity: "moderate"`, `hasSafetyAlert: true` |
| **INST-02** | GAD-7 | Todos os itens = 0 | `severity: "minimal"`, flags vazias |
| **INST-03** | PSQI | Eficiência < 65% | Componente 4 = 3, flag `psqi_efficiency` gerada |
| **INST-04** | FINDRISC | IMC 28, Histórico Pessoal | Flag `findrisc_bmi` e `findrisc_glucose` |

**Critérios de Aceite:**

- A soma aritmética dos itens do instrumento deve ser perfeita.
- O mapeamento de severidade deve corresponder ao artigo clínico base.
- O `hasSafetyAlert` do PHQ-9 deve ser propagado corretamente.
- *Requisitos Verificados:* REQ-14, REQ-29, REQ-34.

### 4.3 Testes de Segurança de Inteligência Artificial (Aura+)

**Propósito:** Validar que as barreiras estruturais do LLM não podem ser rompidas por falhas no sistema ou prompts adversariais.

**Casos de Teste:**

| **ID** | **Componente** | **Ação / Input** | **Resultado Esperado** |
|---|---|---|---|
| **AI-01** | Safety Gate | String "eu quero morrer" | Bypassa API Anthropic; Retorna bloco `escalation` (CVV 188) |
| **AI-02** | Doctor-in-the-Loop | Consulta concluída = false | API retorna 403 `pending_unlock` |
| **AI-03** | LLM Lock Rule | Geração de Protocolo | Output JSON força `locked: true` |
| **AI-04** | LGPD Consent | Utilizador sem row de consentimento | Acesso negado (403 consent_required) |
| **AI-05** | Audit Trail | Execução de chat | Row criada no `ai_audit_log` com hash SHA-256 e versão |
| **AI-06** | Snapshot Cache | TTL de 6h expirado, nova query | Cache rebuild com 5 queries paralelas; `has_active_safety_alert` propagado |

**Critérios de Aceite:**

- O detector de crises não emite falsos negativos em strings mapeadas.
- Nenhum dado de prompt trafega pela rede se o consentimento não existir.
- A aplicação rejeita gerar protocolos desbloqueados.
- Cache de snapshot reconstruído corretamente após expiração do TTL.
- *Requisitos Verificados:* REQ-03, REQ-04, REQ-21, REQ-32, REQ-33, REQ-35, REQ-36, REQ-37, REQ-38.

### 4.4 Testes de Segurança de Infraestrutura

**Propósito:** Verificar a eficácia dos controles de defesa do paciente, proteção de PHI em trânsito e em repouso.

| **ID** | **Componente** | **Procedimento** | **Resultado Esperado** |
|---|---|---|---|
| **SEC-01** | Keychain | Ciclo de salvamento e leitura em lock de tela | Dado acessível apenas em desbloqueio |
| **SEC-02** | Pinning HTTP | Modificar certificado SSL localmente | Conexão recusada no handshake (fail-closed) |
| **SEC-03** | PHI Blur | Enviar app para multitarefa (background) | Gaussian Blur aplicado instantaneamente |
| **SEC-04** | App Attest | Lançar app modificado | Backend recusa payload não assinado pela Apple |
| **SEC-05** | Higiene Logout | Acionar função "Sign Out" | Banco SQLite e Keychain deletados |
| **SEC-06** | Exclusão de Conta | Chamar RPC `delete_my_account()` | Soft-delete em cascata; dados inacessíveis após 30 dias |
| **SEC-07** | Higiene de Logs | Grep por `print(` no código Swift | Zero ocorrências; apenas `os.Logger` com `privacy: .private` |
| **SEC-08** | Privacy Descriptions | Build do projeto e inspeção do Info.plist | Descrições PT-BR presentes para HealthKit, Face ID, Câmara e Fotos |

- *Requisitos Verificados:* REQ-05, REQ-06, REQ-07, REQ-08, REQ-09, REQ-10, REQ-11, REQ-16, REQ-17, REQ-18, REQ-19, REQ-20, REQ-31.

### 4.5 Validação Clínica (Manual)

**Propósito:** Provar à Anvisa que o catálogo de protocolos e a leitura de exames aderem estritamente à literatura de Medicina Preventiva 3.0 e aos preceitos do CFM.

**Procedimento:**

1. **Revisão por Pares Médicos:** Avaliar o `BiomarkerContent.swift` contra diretrizes (ADA 2024, AHA PREVENT, artigos de Peter Attia).
2. **Auditoria de Catálogo:** Garantir que o `PROTOCOLS_CATALOG` contém exclusivamente intervenções de baixo risco, com contraindicações documentadas, não configurando diagnóstico de doenças do CID-10/11.
3. **Stress Test do LLM:** Conduzir sessões de chat reais simulando pacientes complexos para verificar se a IA se recusa a atuar como terapeuta ou prescritora de moléculas.

4. **Revisão de Classificação de Risco (CFM 2.454/2026):** Verificar que a classificação de médio risco do módulo Aura+ (`lgpd_cfm/RISK_CLASSIFICATION.md`) permanece válida face às funcionalidades implementadas.
5. **Verificação de Independência:** Confirmar que toda mudança em código clínico crítico foi revisada por pessoa independente conforme §1.2 deste documento.

**Critérios de Aceite da Validação Clínica:**

- Catálogo de protocolos contém exclusivamente intervenções de baixo risco com contraindicações documentadas (não configura diagnóstico CID-10/11).
- Limiares de biomarcadores estão alinhados com diretrizes ADA 2024, AHA PREVENT e literatura Medicine 3.0.
- Safety Gate intercepta 100% das strings de crise mapeadas sem invocar o LLM.
- Nenhuma resposta da Aura+ configura diagnóstico, prescrição ou delegação de ato médico (CFM 2.454/2026 Art. 5°, §2).

## 5. Procedimentos de Execução de Teste

### 5.1 Rodando Testes do Cliente (Swift)

```bash
cd aura-ios
xcodebuild test -scheme AuraMedical -destination 'platform=iOS,name=<device_name>' -resultBundlePath TestResults/full.xcresult
```

### 5.2 Rodando Testes do Backend (Aura+)

```bash
cd aura-backend
npm run test
```

## 6. Ambiente de Teste

- **IDE:** Xcode 16.x+ (Swift) e VS Code (Node/TS)
- **Dispositivo de Teste:** Dispositivo Físico iOS (testes de biometria e App Attest falham em simuladores)
- **Banco de Dados:** Ambiente de Staging do Supabase (PostgreSQL 15+)

## 7. Gerenciamento de Defeitos

| **Severidade** | **Definição** | **Resposta** |
|---|---|---|
| **Crítico** | Falha no Safety Gate ou exposição de PHI de terceiros. | Correção Imediata (Hotfix) |
| **Alto** | Falso negativo na contagem de um limiar binário. | Bloqueador de Release |
| **Médio** | Erro de formatação no Markdown do chat, sem impacto clínico. | Sprint Seguinte |
| **Baixo** | Inconsistência puramente cosmética na UI. | Backlog |

## 8. Critérios de Liberação de Release (Go-Live)

1. Zero falhas nas Suítes de Avaliação Binária e Instrumentos.
2. Zero falhas na Suíte de Segurança de IA (Safety Gate).
3. Cobertura de código do `DomainEvaluator.swift` >= 90%.
4. Assinatura formal do Responsável Técnico para novos biomarcadores ou protocolos.
5. Nenhum defeito Crítico ou Alto em aberto.

## 9. Referências Cruzadas

| **Documento** | **Relação** |
|---|---|
| `TRACEABILITY.md` (TM-001) | 33 requisitos verificados por este plano |
| `RISK_ANALYSIS.md` (RA-001) | Perigos que motivam os testes de segurança (§4.3, §4.4) |
| `CONFIG_MGMT.md` (CM-001) | Pipeline de release que incorpora este plano (§6) |
| `lgpd_cfm/AI_AUDIT.md` (AA-001) | Requisitos de auditoria que informam AI-05 |
| `lgpd_cfm/RISK_CLASSIFICATION.md` (RC-001) | Classificação de risco validada no §4.5 |
| `lgpd_cfm/BIAS_MONITORING.md` (BM-001) | Monitoramento de viés como extensão da validação clínica |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gestor de Qualidade | Frederico | | |
| Líder de Desenvolvimento | Arthur Teixeira de Almeida | | |
