# Plano de Gerenciamento de Configuração

**ID do Documento:** CM-001

**Revisão:** 3.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Backend

**Norma:** IEC 62304:2006+AMD1:2015 Seção 5.1.9 (Gerenciamento de configuração de software)

---

## 1. Propósito

Este documento define a estratégia de gerenciamento de configuração para o sistema Aura Medical (SaMD), incluindo controle de versão, arquitetura de repositórios, procedimentos de controle de mudança, processos de build e gerenciamento de releases.

## 2. Arquitetura de Repositórios

O sistema Aura Medical é composto por dois repositórios principais que formam o produto final:

```
aura-wearables/
  |-- aura-ios/            # App Swift iOS (Motor de Avaliação Clínica Binária)
  |-- aura-backend/        # API Hono/Node (Motor de IA, Gateways e Supabase)
```

### 2.1 Papéis dos Repositórios

| **Repositório** | **Linguagem** | **Papel no Sistema** | **Componente Crítico** |
|---|---|---|---|
| `aura-ios` | Swift | Frontend e fonte da verdade da lógica clínica (Medicine 3.0). | `DomainEvaluator.swift`, `InstrumentScoring.swift` |
| `aura-backend` | TypeScript | Orquestração de IA, segurança e persistência de dados. | `system-prompt.ts`, `safety.ts`, `audit.ts` |

## 3. Esquema de Versionamento

O sistema adota o Versionamento Semântico (SemVer) e regras rígidas de rastreabilidade para o modelo de Inteligência Artificial:

### 3.1 Versão do Software (App e Backend)

| **Componente** | **Formato** | **Fonte da Verdade** | **Exemplo** |
|---|---|---|---|
| **iOS App** | `X.Y.Z` | `project.yml` / Xcode | `1.0.0` |
| **Backend** | `X.Y.Z` | `package.json` | `1.0.0` |

**Regras de Versão:**

- **Major (X):** Mudanças arquiteturais ou adição de novos domínios clínicos.
- **Minor (Y):** Novas features e instrumentos de avaliação (retrocompatíveis).
- **Patch (Z):** Correções de bugs, ajustes de limiares (thresholds) e copy.

### 3.2 Versão do Modelo de IA (Rastreabilidade Exata)

Para fins de auditoria (CFM 2.314), é proibido o uso de aliases genéricos para o LLM. A versão do modelo deve ser fixada em um snapshot datado no código:

- **Fonte:** `aura-backend/src/health/aura-plus/anthropic.ts`
- **Exemplo Atual:** `claude-haiku-4-5-20251001`

## 4. Controle de Mudanças (Change Control)

### 4.1 Categorias de Mudança

Qualquer alteração no código-fonte é classificada e exige aprovações específicas antes do merge para a branch principal (`main`):

| **Categoria** | **Escopo** | **Aprovação Exigida** | **Teste Exigido** |
|---|---|---|---|
| **Crítica** | Lógica de `DomainEvaluator.swift`, prompts de IA ou Safety Gates (`safety.ts`). | RT (Médico) + QA Lead | Testes Unitários Completos |
| **Padrão** | Controles de segurança, sync de dados (SwiftData/Supabase). | 1 Revisor (Dev Sênior) | Testes de Integração |
| **Menor** | UI/UX, textos, documentação (sem impacto clínico). | 1 Revisor | QA Visual / Manual |
| **Emergência** | Vulnerabilidade de segurança ou falha crítica em produção. | Revisão Pós-incidente permitida | Testes Mínimos Viáveis |

### 4.2 Regras para Código Clínico e de Segurança

- **Lógica Clínica (`DomainEvaluator.swift`, `InstrumentScoring.swift`):** Qualquer alteração nos limiares de exames ou lógicas de pontuação exige validação clínica formal pelo Responsável Técnico.
- **Prompt da IA (`system-prompt.ts`):** Alterações nas regras ou anti-padrões da Aura+ devem ser testadas contra regressão de alucinação e revisadas em conjunto.
- **Segurança (`AuraMedical/Security/`):** É estritamente proibido o uso de comandos `print()` para dados. Todo log deve utilizar `os.Logger` com anotação `privacy: .private` para proteção de PHI.

## 5. Processo de Build

### 5.1 Build iOS (Cliente)

A infraestrutura de build do iOS é gerada dinamicamente para evitar conflitos de projeto:

| **Passo** | **Comando** | **Descrição** |
|---|---|---|
| 1 | `xcodegen generate` | Gera o `AuraMedical.xcodeproj` a partir do `project.yml`. |
| 2 | `xcodebuild build` | Compila o scheme `AuraMedical` (Debug/Release). |
| 3 | `xcodebuild test` | Executa a suíte de testes unitários. |
| 4 | `xcodebuild archive` | Cria o arquivo de release para assinatura. |

- **Identificador:** `med.aura.medical`
- **Team ID:** RZ9WFC8ZMG (Auramedical Tecnologia)

### 5.2 Build Backend

| **Passo** | **Comando** | **Descrição** |
|---|---|---|
| 1 | `npm install` | Instalação de dependências estritas. |
| 2 | `npm run build` | Compilação do TypeScript (Hono). |
| 3 | `npm start` | Inicialização do servidor de produção. |

## 6. Processo de Release

O lançamento de novas versões obedece ao seguinte pipeline para garantir o compliance de SaMD:

1. **Feature Freeze:** Congelamento da branch de release.
2. **Version Bump:** Atualização das versões no `project.yml` e `package.json`.
3. **Test Pass:** Execução e aprovação de 100% dos testes unitários e de integração.
4. **Aprovação Clínica:** Assinatura do RT caso existam mudanças no avaliador de domínio.
5. **Git Tag:** Criação de tag imutável no repositório (ex: `v1.0.0`).
6. **Distribuição Interna:** TestFlight para QA regression (mínimo de 3 dias).
7. **Submissão:** Envio para aprovação da App Store e deploy de produção do Backend.

## 7. Ferramentas Homologadas

| **Ferramenta** | **Propósito** | **Ambiente / Versão** |
|---|---|---|
| **Xcode / Swift** | IDE e Compilação iOS | 16.x ou superior |
| **xcodegen** | Geração determinística de projeto iOS | Última versão estável |
| **Git / GitHub** | Controle de versão e revisão de PRs | Nuvem |
| **Node.js / npm** | Runtime e gerenciamento do Backend | LTS (v20+) |
| **Supabase** | BaaS, Banco de Dados, Auth, Logs | Nuvem |
| **TestFlight** | Distribuição de QA e homologação | Nuvem |

## 8. Trilha de Auditoria (Audit Trail)

Todas as mudanças de configuração e execuções do sistema são rastreáveis através de:

1. **Histórico Git:** Commits imutáveis vinculados a autores.
2. **Pull Requests:** Aprovações formais documentadas no GitHub.
3. **Logs de IA:** Tabela `ai_audit_log` no Supabase armazenando versões do LLM e hashes criptográficos de cada interação de suporte à decisão.
4. **Tags de Release:** Marcações no Git garantindo a reprodução exata de qualquer build em produção.

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Compliance e Qualidade | Frederico | | |
