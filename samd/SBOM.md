# Software Bill of Materials (SBOM)

**ID do Documento:** SB-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend (v1.0.0)

**Norma:** IEC 62304:2006+AMD1:2015 §5.3 (Componentes de software de procedência externa — SOUP)

---

## 1. Propósito

Este documento lista todos os componentes de software de procedência externa (SOUP — Software of Unknown Provenance, conforme IEC 62304 §5.3) utilizados no produto Aura Medical v1.0.0, incluindo versões fixas, licenças, propósito e avaliação de risco.

## 2. Backend — Aura+ (TypeScript / Node.js)

**Runtime:** Node.js LTS v20+

**Fonte de Verdade:** `aura-backend/package.json`

### 2.1 Dependências de Produção

| Pacote | Versão Fixada | Licença | Propósito no SaMD | Impacto Clínico |
|---|---|---|---|---|
| `@anthropic-ai/sdk` | 0.32.1 | MIT | Cliente da API Anthropic (LLM Claude Haiku). Motor de IA conversacional do Aura+. | **Alto** — gera conteúdo apresentado ao utilizador. Mitigado por guardrails e Safety Gate. |
| `@google/generative-ai` | ^0.24.0 | Apache-2.0 | OCR de exames laboratoriais via Gemini Flash (lab-parse.ts). | **Médio** — processa dados de entrada. Validado por filtro de biomarcadores aceitos. |
| `@hono/node-server` | ^1.19.11 | MIT | Adaptador Node.js para o framework Hono. | Baixo — infraestrutura HTTP. |
| `hono` | 4.12.9 | MIT | Framework HTTP/SSE do backend (rotas, middleware, streaming). | Baixo — infraestrutura. |
| `@supabase/supabase-js` | 2.100.1 | MIT | Cliente PostgreSQL/Auth (RLS, queries, auth JWT). | **Médio** — interface com banco de dados de saúde. Protegido por RLS. |
| `pino` | ^10.3.1 | MIT | Logger estruturado (JSON). Substitui `console.log` com `privacy: .private`. | Baixo — observabilidade. |
| `zod` | ^4.3.6 | MIT | Validação de schema para inputs de API e variáveis de ambiente. | Baixo — defesa de input. |

### 2.2 Dependências de Desenvolvimento (não entram em produção)

| Pacote | Versão | Propósito |
|---|---|---|
| `@types/node` | ^25.5.0 | Type definitions para Node.js |
| `tsx` | ^4.21.0 | Runner TypeScript para desenvolvimento |
| `typescript` | ^6.0.2 | Compilador TypeScript |
| `vitest` | ^4.1.2 | Framework de testes unitários e integração |

## 3. iOS — Aura Medical App (Swift)

**Plataforma mínima:** iOS 17

**Fonte de Verdade:** `aura-ios/Package.swift`

### 3.1 Dependências de Produção

| Pacote | Versão | Licença | Propósito no SaMD | Impacto Clínico |
|---|---|---|---|---|
| `supabase-swift` | ≥ 2.0.0 | MIT | Cliente Supabase para iOS (Auth, Realtime, Storage). | **Médio** — interface com banco de dados de saúde. |

### 3.2 Frameworks do Sistema (Apple SDK)

| Framework | Propósito | Impacto Clínico |
|---|---|---|
| `HealthKit` | Ingestão de dados de wearables (HRV, sono, passos, FC). | **Alto** — dados de entrada clínica. Validados pelo `HealthSource` struct. |
| `SwiftData` | Persistência local (cache offline). | Médio — armazenamento PHI local. Protegido por `NSFileProtectionComplete`. |
| `LocalAuthentication` | Biometria (Face ID / Touch ID). | Médio — controle de acesso. |
| `CryptoKit` | Operações criptográficas (hashing, assinatura). | Baixo — infraestrutura de segurança. |
| `DeviceCheck` | App Attest (integridade do dispositivo). | Baixo — defesa contra tampering. |

## 4. Modelo de IA (Subprocessador)

| Componente | Provedor | Versão | Propósito | Impacto |
|---|---|---|---|---|
| Claude Haiku | Anthropic, Inc. (EUA) | `claude-haiku-4-5-20251001` (snapshot datado) | LLM conversacional do módulo Aura+. | **Alto** — gera conteúdo clínico educacional. |
| Gemini 2.5 Flash | Google (EUA) | Via `@google/generative-ai` | OCR de exames laboratoriais. | **Médio** — extrai valores de biomarcadores de PDFs. |

**Nota:** Aliases genéricos (`claude-haiku-latest`) são proibidos para fins de auditabilidade (CFM 2.454/2026 Anexo I, XVIII). A versão do modelo é fixada no código-fonte (`anthropic.ts`) e registrada em cada interação no `ai_audit_log`.

## 5. Infraestrutura de Serviço

| Serviço | Provedor | Propósito | Dados Processados |
|---|---|---|---|
| Supabase | Supabase Inc. | BaaS (PostgreSQL, Auth, Storage, Edge Functions) | PHI (dados de saúde, perfil, biomarcadores) |
| Anthropic API | Anthropic, Inc. | LLM as a Service | Snapshot de saúde do utilizador (via system prompt) |
| Google AI API | Google LLC | OCR de exames | Imagens de exames laboratoriais |
| Apple App Store | Apple Inc. | Distribuição do app iOS | Metadados da app |
| TestFlight | Apple Inc. | Distribuição de QA | Builds de teste |

## 6. Avaliação de Risco SOUP (IEC 62304 §5.3.3)

Conforme IEC 62304 §5.3.3, cada SOUP de impacto Alto ou Médio deve ser avaliado quanto ao risco de falha:

| SOUP | Risco de Falha | Mitigação |
|---|---|---|
| `@anthropic-ai/sdk` | Alucinação do LLM | Safety Gate pré-LLM + guardrails no system prompt + Doctor-in-the-Loop |
| `@google/generative-ai` | OCR incorreto | Filtro server-side de biomarcadores aceitos; valores absurdos rejeitados |
| `@supabase/supabase-js` | Falha de autenticação ou RLS | RLS em todas as tabelas; JWT validado em cada request |
| `supabase-swift` | Idem | Idem |
| `HealthKit` | Dados de wearable corrompidos | `HealthSource` valida bundle ID; Ghost Mode para dados insuficientes |
| Claude Haiku (LLM) | Saídas inadequadas | 3 controles críticos: Safety Gate + Doctor-in-the-Loop + Audit Trail |
| Gemini Flash (OCR) | Extração incorreta | Lista whitelist de biomarcadores aceitos; rate limiting |

## 7. Política de Atualização

- **Dependências de produção:** Atualizadas apenas via Change Control (CONFIG_MGMT.md §4). Categoria "Padrão" para patches de segurança; "Crítica" para mudanças de versão major.
- **Modelo de IA:** Mudança de snapshot do LLM exige aprovação do RT e da Comissão de IA (AI_GOVERNANCE.md §4.4).
- **Revisão periódica:** SBOM revisado a cada release ou quando CVEs relevantes são publicados.

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Compliance e Qualidade | Frederico | | |
