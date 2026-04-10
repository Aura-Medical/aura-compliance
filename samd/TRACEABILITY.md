# Matriz de Rastreabilidade de Requisitos

**ID do Documento:** TM-001

**Revisão:** 3.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend

**Norma:** IEC 62304:2006+AMD1:2015 Seção 5.1.1 (Planeamento do desenvolvimento de software)

---

## 1. Propósito

Este documento estabelece a rastreabilidade bidirecional entre requisitos de software, especificações de design, implementações no código-fonte, procedimentos de teste e evidências de verificação. Cada requisito é vinculado ao(s) perigo(s) correspondente(s) definidos em `RISK_ANALYSIS.md` (RA-001 Rev 3.0) onde aplicável.

## 2. Convenções

- **Caminhos de código-fonte** são relativos às raízes dos repositórios (`aura-ios/` e `aura-backend/`).
- **Status de teste** indica o estado atual de verificação: Implementado, Planeado ou Pendente.
- **Referências de perigo** vinculam-se aos identificadores HAZ-01 a HAZ-17 definidos em `RISK_ANALYSIS.md`.
- **IDs consolidados:** Os IDs REQ-22, REQ-23, REQ-26, REQ-27 e REQ-30 foram consolidados em outros requisitos durante a migração arquitetural para a v1.0.0 e não são mais utilizados.

---

## 3. Tabela de Rastreabilidade

### REQ-01: Validação de Inputs e Limites Fisiológicos

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-01 |
| **Descrição** | Todos os inputs numéricos (biomarcadores, wearables) devem ser validados contra limites fisiológicos. Valores extremos devem ser sinalizados ou rejeitados para impedir que dados corrompidos distorçam a avaliação clínica. |
| **Design** | `BiomarkerContent` define explicações clínicas e limiares. `DomainEvaluator` verifica limites rígidos específicos (ex: ApoB > 90, Glicose >= 100) antes de sinalizar um item. |
| **Código-Fonte** | `AuraMedical/DomainEvaluator.swift:evaluateDomain()`, `AuraMedical/BiomarkerContent.swift` |
| **Teste** | Vetores de teste com inputs fora dos limites; testes unitários para cada verificação de limiar de biomarcador. |
| **Evidência** | Logs de testes unitários verificando que entradas extremas são tratadas sem crash ou falsa tranquilização. |
| **Vínculo de Perigo** | HAZ-01, HAZ-08 |
| **Status** | Implementado |

### REQ-02: Avaliação Binária de Cinco Domínios (Modelo de Seeman)

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-02 |
| **Descrição** | O sistema deve avaliar exatamente cinco domínios de saúde: metabólico, cardiovascular, sono, nervoso (mente) e movimento, utilizando um verificador de limiares binários (0 = saudável, 1 = sinalizado) em vez de regressões ponderadas. |
| **Design** | `DomainEvaluator.evaluateDomain` mapeia perguntas e biomarcadores específicos a domínios. Se `value >= threshold`, o item é adicionado ao array `flaggedItems`. |
| **Código-Fonte** | `AuraMedical/DomainEvaluator.swift:isItemFlagged()`, `AuraMedical/DomainEvaluator.swift:domainQuestions` |
| **Teste** | Testes unitários mapeando outputs de questionários para contagens corretas de flags por domínio. |
| **Evidência** | Vetores de avaliação verificando que exatamente cinco objetos `DomainStatus` são retornados. |
| **Vínculo de Perigo** | HAZ-01, HAZ-06 |
| **Status** | Implementado |

### REQ-03: Trilha de Auditoria de IA para SaMD (CFM 2.314)

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-03 |
| **Descrição** | Toda computação de IA (chat ou opener) deve ser registrada para auditabilidade regulatória (CFM 2.314 Art. 5). O log deve incluir ID do utilizador, ID da sessão, versão do modelo e hashes SHA-256 do prompt e da resposta exatos (compatível com LGPD). |
| **Design** | `logAiInteraction` insere registros em `ai_audit_log` usando abordagem fail-soft para evitar bloqueio da resposta ao utilizador. Prompts reais são hashados com `crypto.createHash('sha256')`. |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/audit.ts`, `aura-backend/src/health/aura-plus/chat.ts` |
| **Teste** | Teste de integração: verificar row criada após interação com LLM; verificar não-bloqueio em timeout do Supabase. |
| **Evidência** | Tabela `ai_audit_log` no Supabase. |
| **Vínculo de Perigo** | HAZ-05, HAZ-12 |
| **Status** | Implementado |

### REQ-04: Fallback Determinístico e Bypass de Crise

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-04 |
| **Descrição** | O sistema deve detectar proativamente inputs de crise (ex: ideação suicida) usando padrões de regex determinísticos. Se detectado, o LLM deve ser bypassado e um bloco de escalonamento crítico (CVV 188 / SAMU 192) deve ser retornado. |
| **Design** | `isCrisisInput` executa antes do gate do LLM. Se houver match, `crisisEscalationBlock` é enviado diretamente via SSE com severidade `critical` e `safety-bypass` como nome do modelo. |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/safety.ts`, `aura-backend/src/health/aura-plus/chat.ts` |
| **Teste** | Testes unitários de backend passando strings de crise conhecidas para verificar escalonamento imediato sem invocação da API Anthropic. |
| **Evidência** | Padrões regex em `safety.ts` e fluxo de bypass em `chat.ts`. |
| **Vínculo de Perigo** | HAZ-02, HAZ-03 |
| **Status** | Implementado |

### REQ-05: Proteção de PHI em Background

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-05 |
| **Descrição** | Todas as telas que exibem PHI devem aplicar um Gaussian Blur de raio 30 quando o app entra em background, está sendo gravado em tela ou capturado. |
| **Design** | `PHIProtectionModifier` observa `scenePhase` e `UIScreen.capturedDidChangeNotification`. Aplica blur quando `scenePhase != .active` ou `isCaptured == true`. |
| **Código-Fonte** | `AuraMedical/Security/PHIProtectionModifier.swift` |
| **Teste** | QA Manual: verificar blur ao trocar de app, gravação de tela e screenshot. |
| **Evidência** | Aplicado em BioHubView, DomainDetailView, PersonalDataView, ScoreView. |
| **Vínculo de Perigo** | HAZ-07, HAZ-10 |
| **Status** | Implementado |

### REQ-06: Autenticação Biométrica

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-06 |
| **Descrição** | Quando habilitado pelo utilizador, Face ID / Touch ID deve ser obrigatório para aceder ao app. Fallback para passcode do dispositivo. Estado de autenticação armazenado em Keychain (não UserDefaults). |
| **Design** | `BiometricAuthManager` (@Observable) usa `LAContext.evaluatePolicy(.deviceOwnerAuthentication)`. Habilitação/desabilitação persistida em Keychain via `AuraKeychain`. |
| **Código-Fonte** | `AuraMedical/Security/BiometricAuthManager.swift` |
| **Teste** | QA Manual: habilitar bloqueio biométrico, verificar que tela de bloqueio aparece ao retomar o app. |
| **Evidência** | Entrada de Keychain `biometricAuth.enabled`. |
| **Vínculo de Perigo** | HAZ-07, HAZ-10 |
| **Status** | Implementado |

### REQ-07: Certificate Pinning

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-07 |
| **Descrição** | Todas as conexões aos hosts Supabase e Backend devem utilizar certificate pinning SPKI SHA-256 com política fail-closed. Pin de backup para CA intermediária deve ser incluído para resiliência de rotação. |
| **Design** | `PinnedURLSessionDelegate` extrai a chave pública do certificado do servidor, prepende o header ASN.1 correto, computa SHA-256 e compara contra o conjunto de hashes fixos. Rejeita conexão se não houver match. |
| **Código-Fonte** | `AuraMedical/Security/CertificatePinning.swift` |
| **Teste** | Teste de integração: verificar sucesso de conexão para hosts válidos; verificar rejeição em mismatch de pin. |
| **Evidência** | Hosts fixos configurados na instância URLSession compartilhada. |
| **Vínculo de Perigo** | HAZ-06 |
| **Status** | Implementado |

### REQ-08: Timeout de Inatividade de Sessão

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-08 |
| **Descrição** | Quando a autenticação biométrica estiver habilitada, o app deve auto-bloquear após 30 minutos de inatividade (sem interação do utilizador). |
| **Design** | `BiometricAuthManager` rastreia `lastInteractionDate` e executa `idleTimer` que verifica o tempo decorrido contra `idleTimeoutInterval` (30 * 60 segundos). |
| **Código-Fonte** | `AuraMedical/Security/BiometricAuthManager.swift` |
| **Teste** | QA Manual: aguardar 30 minutos, verificar que tela de bloqueio aparece. |
| **Evidência** | `BiometricAuthManager.idleTimeoutInterval = 30 * 60`. |
| **Vínculo de Perigo** | HAZ-07, HAZ-10 |
| **Status** | Implementado |

### REQ-09: Armazenamento Seguro em Keychain

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-09 |
| **Descrição** | Todos os dados sensíveis (tokens, chaves) devem ser armazenados no iOS Keychain com acessibilidade `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`. Dados não devem migrar para backups ou novos dispositivos. |
| **Design** | Enum `AuraKeychain` fornece API tipada de save/load/delete. Método `deleteAll()` limpa todas as entradas do app no logout. |
| **Código-Fonte** | `AuraMedical/Security/AuraKeychain.swift` |
| **Teste** | Teste unitário: ciclo save/load/delete; verificar atributo de acessibilidade na query. |
| **Evidência** | Identificador de serviço Keychain `med.aura.medical`. |
| **Vínculo de Perigo** | HAZ-07, HAZ-10, HAZ-15 |
| **Status** | Implementado |

### REQ-10: Sincronização Offline-First

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-10 |
| **Descrição** | O app deve funcionar offline utilizando dados persistidos localmente (SwiftData). Escritas vão para o local imediatamente com flag `needsUpload`, sincronizadas com o servidor assincronamente. |
| **Design** | `SyncManager` coordena sync SwiftData ↔ Supabase. ModelContainer usa `NSFileProtectionComplete` para criptografia em repouso. |
| **Código-Fonte** | `AuraMedical/LocalData/SyncManager.swift` |
| **Teste** | Teste de integração: verificar leitura local sem rede; verificar sync após reconexão. |
| **Evidência** | Modelos SwiftData `LocalScore`, `LocalProfile`, `LocalBiometrics`. |
| **Vínculo de Perigo** | HAZ-06, HAZ-11 |
| **Status** | Implementado |

### REQ-11: Integridade de Dispositivo via App Attest

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-11 |
| **Descrição** | O app deve utilizar Apple App Attest para gerar e armazenar uma chave de attestation do dispositivo no primeiro lançamento. Key ID armazenado em Keychain. |
| **Design** | Actor `AppAttestManager` usa `DCAppAttestService` para gerar chave. `attest()` produz attestation para hash de challenge fornecido pelo servidor. |
| **Código-Fonte** | `AuraMedical/Security/AppAttestManager.swift` |
| **Teste** | Teste em dispositivo: verificar geração de chave em dispositivo suportado. |
| **Evidência** | Entrada de Keychain `appAttest.keyId`. |
| **Vínculo de Perigo** | HAZ-10, HAZ-15 |
| **Status** | Implementado |

### REQ-12: Data Caps de Biomarcadores Críticos

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-12 |
| **Descrição** | Biomarcadores com valores acima dos limiares de alerta clínico devem forçar imediatamente a flag do domínio correspondente, independentemente de quaisquer outros inputs positivos. Este mecanismo ("Data Cap") impede que resultados positivos de estilo de vida ocultem riscos laboratoriais críticos. |
| **Design** | `DomainEvaluator.evaluateDomain()` verifica limites rígidos específicos (ApoB > 90 mg/dL, Glicose >= 100 mg/dL, HbA1c >= 5.7%, PCR-us > 3.0 mg/L) e força a flag do domínio imediatamente, antes de avaliar os demais inputs. |
| **Código-Fonte** | `AuraMedical/DomainEvaluator.swift:evaluateDomain()` |
| **Teste** | EV-03: ApoB = 92 mg/dL → flag `apob_elevated` ativada; EV-05: HbA1c 5.8% + questionário saudável → flag `hba1c_prediabetic` ativada. |
| **Evidência** | Lógica de "cap" no `evaluateDomain` demonstrando que o input laboratorial sobrepõe o questionário. |
| **Vínculo de Perigo** | HAZ-01 |
| **Status** | Implementado |

### REQ-13: Ghost Mode (Fallback para Dados Insuficientes)

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-13 |
| **Descrição** | Quando categorias de dados estiverem ausentes, os domínios devem aplicar estimativas conservadoras ou interromper a avaliação para prevenir falsos positivos de saúde. |
| **Design** | Um domínio exige `minimumItemsForStatus = 3`. Se os inputs estiverem abaixo deste limiar, o motor retorna um nível nulo com uma narrativa solicitando mais dados. |
| **Código-Fonte** | `AuraMedical/DomainEvaluator.swift:evaluateDomain()` |
| **Teste** | Teste unitário fornecendo < 3 pontos de dados por domínio. |
| **Evidência** | `DomainStatus` retornado com `level = nil`. |
| **Vínculo de Perigo** | HAZ-01, HAZ-11 |
| **Status** | Implementado |

### REQ-14: Integração de Instrumentos Clínicos Validados

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-14 |
| **Descrição** | Instrumentos clínicos padronizados (PHQ-9, GAD-7, PSQI, FINDRISC) devem ser avaliados conforme diretrizes clínicas e gerar flags de risco específicas. |
| **Design** | Estruturas de pontuação dedicadas (ex: `scorePHQ9`, `scorePSQI`) mapeiam respostas ordinais para tiers de severidade e substituem flags básicas do onboarding (`replacesFlags`) por flags derivadas de maior confiança (`derivedFlags`). |
| **Código-Fonte** | `AuraMedical/InstrumentScoring.swift` |
| **Teste** | Testes unitários comparando vetores de score clínico conhecidos com as flags derivadas corretas. |
| **Evidência** | Array `derivedFlags` substituindo `flaggedItems` genéricos. |
| **Vínculo de Perigo** | HAZ-11 |
| **Status** | Implementado |

### REQ-15: Validação de Dados de Wearables Externos

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-15 |
| **Descrição** | Dados provenientes de wearables externos devem ser validados quanto à fonte (bundle identifier) e formato antes de serem utilizados na avaliação clínica. Dados de fontes desconhecidas devem ser descartados. |
| **Design** | Enum `HealthSource` mapeia bundle identifiers conhecidos (Garmin, Oura, Whoop, Apple Watch) para ícones e tipos de dados específicos. `HealthKitSyncService` valida a origem antes da ingestão. |
| **Código-Fonte** | `AuraMedical/HealthDataModels.swift`, `AuraMedical/HealthKitSyncService.swift` |
| **Teste** | Teste com mock de fontes de wearable: injeção de bundle ID desconhecido deve ser ignorada. |
| **Evidência** | Enum `HealthSource` processando strings de `bid`. |
| **Vínculo de Perigo** | HAZ-01, HAZ-17 |
| **Status** | Implementado |

### REQ-16: Criptografia de Dados em Repouso

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-16 |
| **Descrição** | Todos os dados de saúde persistidos localmente devem ser criptografados em repouso utilizando iOS file protection. O armazenamento SwiftData deve usar `NSFileProtectionComplete`. |
| **Design** | `SyncManager.makeContainer()` cria `ModelContainer` com `NSFileProtectionComplete`. |
| **Código-Fonte** | `AuraMedical/LocalData/SyncManager.swift` |
| **Teste** | Verificar configuração do ModelContainer. |
| **Evidência** | Caminho do SwiftData store com atributo de proteção. |
| **Vínculo de Perigo** | HAZ-08, HAZ-15 |
| **Status** | Implementado |

### REQ-17: Exclusão de Conta

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-17 |
| **Descrição** | Utilizadores devem poder excluir sua conta. RPC do lado do servidor realiza exclusão em cascata em todas as tabelas com retenção soft-delete de 30 dias. |
| **Design** | Função RPC `delete_my_account()` do Supabase realiza soft-delete. |
| **Código-Fonte** | Supabase RPC `delete_my_account()` |
| **Teste** | Teste de integração: chamar RPC, verificar flag de soft-delete ativada. |
| **Evidência** | Definição da função Supabase. |
| **Vínculo de Perigo** | HAZ-13 |
| **Status** | Implementado |

### REQ-18: Logout Limpa Dados Locais

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-18 |
| **Descrição** | No sign-out, todo PHI cacheado localmente deve ser excluído: modelos SwiftData, entradas de Keychain e estado em memória. |
| **Design** | `AuthViewModel.signOut()` chama sign-out do Supabase, exclui modelos via `ModelContext.delete` e chama `AuraKeychain.deleteAll()`. |
| **Código-Fonte** | `AuraMedical/ViewModels/AuthViewModel.swift:signOut()` |
| **Teste** | Teste de integração: sign out, verificar SwiftData vazio, verificar Keychain limpo. |
| **Evidência** | Revisão de código do método `signOut()`. |
| **Vínculo de Perigo** | HAZ-13 |
| **Status** | Implementado |

### REQ-19: Proibição de PHI em Logs

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-19 |
| **Descrição** | Logs da aplicação não devem conter PHI. Todo logging deve usar `os.Logger` com `privacy: .private` ou o logger Pino do backend com redação de campos PHI. |
| **Design** | Valores sensíveis são hashados (SHA-256) ou anotados com `privacy: .private`. |
| **Código-Fonte** | `AuraMedical/Security/` e `aura-backend/src/shared/logger.ts` |
| **Teste** | Revisão de código: grep por `print(`; verificar uso de os.Logger. |
| **Evidência** | Análise estática de padrões de logging. |
| **Vínculo de Perigo** | HAZ-05 |
| **Status** | Implementado |

### REQ-20: Descrições de Uso de Privacidade

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-20 |
| **Descrição** | Info.plist deve incluir descrições de uso para HealthKit, Face ID, Câmara e Biblioteca de Fotos. |
| **Design** | Descrições fornecidas em PT-BR explicando por que cada permissão é necessária. |
| **Código-Fonte** | `project.yml` — seção `info.plist` |
| **Teste** | Verificação de build. |
| **Evidência** | Conformidade com revisão da App Store. |
| **Vínculo de Perigo** | Nenhum (Regulatório) |
| **Status** | Implementado |

### REQ-21: Rastreamento de Versão do LLM

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-21 |
| **Descrição** | A versão do modelo LLM subjacente deve ser explicitamente definida e rastreada em todos os logs de auditoria de IA para garantir rastreabilidade determinística de comportamento. |
| **Design** | `HAIKU_MODEL` é um ID de snapshot datado codificado (ex: `claude-haiku-4-5-20251001`), evitando upgrades silenciosos da API. |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/anthropic.ts:HAIKU_MODEL` |
| **Teste** | Revisão do log de auditoria. |
| **Evidência** | Coluna `ai_audit_log.model` utiliza ID de snapshot. |
| **Vínculo de Perigo** | HAZ-12 |
| **Status** | Implementado |

### REQ-24: Limiares de Referência Específicos por Sexo

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-24 |
| **Descrição** | Faixas de referência para biomarcadores como ácido úrico e HDL devem ser avaliadas condicionalmente com base no sexo biológico do utilizador. |
| **Design** | O avaliador verifica `answers["biological_sex"]` antes de aplicar o limiar (ex: Ácido Úrico limiar 7.0 para masculino, 6.0 para feminino). |
| **Código-Fonte** | `AuraMedical/DomainEvaluator.swift:evaluateDomain()` |
| **Teste** | Vetores de teste para inputs masculinos e femininos. |
| **Evidência** | Branches de avaliação mapeados em `evaluateDomain`. |
| **Vínculo de Perigo** | HAZ-09, HAZ-14 |
| **Status** | Implementado |

### REQ-25: Lógica de Nível de Status (Cinco Tiers)

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-25 |
| **Descrição** | Scores devem ser mapeados para uma escala qualitativa (Nível de Status) para informar o utilizador sem fornecer scores absolutos diagnósticos. |
| **Design** | `StatusLevel.from(flagCount: totalItems)` deriva o estado clínico da proporção de itens sinalizados vs saudáveis em um dado domínio. |
| **Código-Fonte** | `AuraMedical/DomainEvaluator.swift` |
| **Teste** | Teste unitário verificando limites de tiers. |
| **Evidência** | Propriedade `DomainStatus.level`. |
| **Vínculo de Perigo** | HAZ-01 |
| **Status** | Implementado |

### REQ-28: Ingestão Multi-Fonte via HealthKit

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-28 |
| **Descrição** | O app deve ingerir dados de múltiplos ecossistemas de wearables (Garmin, Oura, Whoop, Apple) para construir as avaliações de atividade física e sono. |
| **Design** | `HealthSource` mapeia bundle identifiers para ícones e tipos de dados específicos. `HealthKitSyncService` sincroniza resumos diários em background. |
| **Código-Fonte** | `AuraMedical/HealthDataModels.swift`, `HealthKitSyncService.swift` |
| **Teste** | Teste de injeção de fontes de wearable via mock. |
| **Evidência** | Enum `HealthSource` processando strings de `bid`. |
| **Vínculo de Perigo** | HAZ-11, HAZ-17 |
| **Status** | Implementado |

### REQ-29: Indicador de Confiança / Precisão dos Dados

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-29 |
| **Descrição** | O app deve calcular uma métrica `DataConfidence` refletindo a disponibilidade de wearables, instrumentos validados e dados laboratoriais para informar o utilizador sobre a fiabilidade da avaliação. |
| **Design** | Verificações booleanas para `hasWearable`, `hasValidatedInstrument` e `hasLabs` combinam-se para definir o tier de confiança dos dados da avaliação. |
| **Código-Fonte** | `AuraMedical/DomainEvaluator.swift:DataConfidence` |
| **Teste** | Testes de UI validando o badge de precisão. |
| **Evidência** | Propriedade `DomainStatus.confidence`. |
| **Vínculo de Perigo** | HAZ-01, HAZ-11 |
| **Status** | Implementado |

### REQ-31: Autenticação JWT e RLS no Supabase

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-31 |
| **Descrição** | Todas as chamadas de API autenticadas devem incluir um JWT Supabase válido. O banco de dados DEVE impor Row-Level Security (RLS) para que utilizadores possam estritamente consultar apenas registros do seu próprio UUID. |
| **Design** | `AuraBackendClient` anexa o JWT da sessão. Políticas do Supabase impõem `auth.uid() = user_id` para todas as operações. |
| **Código-Fonte** | `AuraMedical/ViewModels/AuthViewModel.swift`, Políticas RLS do PostgreSQL. |
| **Teste** | Teste de integração: verificar JWT presente; verificar 401/403 em token expirado ou fetch de registro não pertencente ao utilizador. |
| **Evidência** | Middleware de auth do backend e definições de schema do BD. |
| **Vínculo de Perigo** | HAZ-07, HAZ-08, HAZ-10 |
| **Status** | Implementado |

### REQ-32: Cache de Snapshot de Dados (Desempenho/Segurança)

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-32 |
| **Descrição** | Para garantir que respostas do LLM atuem sobre dados clínicos frescos porém estáveis, um User Snapshot é cacheado com TTL de 6 horas, reconstruído a partir de 5 queries paralelas ao Supabase (perfis, questionários, biomarcadores, assessments, wearables) se stale. |
| **Design** | `fetchUserSnapshot` verifica `health_snapshot_cache`. Recomputa e upsert em caso de miss. O snapshot dita restrições do LLM via `has_active_safety_alert`. |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/snapshot.ts` |
| **Teste** | Testes de cache hit/miss simulando passagem de tempo. |
| **Evidência** | Queries ao BD `health_snapshot_cache`. |
| **Vínculo de Perigo** | HAZ-16 |
| **Status** | Implementado |

### REQ-33: System Prompt Contexto-Aware do Backend

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-33 |
| **Descrição** | O system prompt do LLM deve injetar dinamicamente o snapshot clínico do utilizador e protocolos de formatação estritos (XML) enquanto adere às doutrinas Medicine 3.0. |
| **Design** | `buildFullSystemPrompt` integra nome do utilizador, regras de disclaimer, safety gates e a representação Markdown do snapshot para delimitar o contexto de geração da IA. |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/system-prompt.ts` |
| **Teste** | Testes de validação de prompt. |
| **Evidência** | Regra estrita no system prompt: "NUNCA diagnostica". |
| **Vínculo de Perigo** | HAZ-02 |
| **Status** | Implementado |

### REQ-34: Integração de Alertas de Segurança Clínica (Domínio Mente)

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-34 |
| **Descrição** | Se um assessment (ex: PHQ-9 item 9) acionar um alerta de segurança, a flag `has_active_safety_alert` do snapshot torna-se verdadeira por 90 dias. Isto adiciona um adendo crítico ao prompt do LLM para tratar qualquer queixa de sintomas com um bloco de emergência imediato. |
| **Design** | `snapshot.has_active_safety_alert` adiciona condicionalmente uma seção `# SAFETY GATE ATIVO` ao prompt ditando resposta de bloco de escalonamento. |
| **Código-Fonte** | `snapshot.ts`, `system-prompt.ts` |
| **Teste** | Backend e2e com snapshot sinalizado acionando o bloco de escalonamento. |
| **Evidência** | Avaliação de output do prompt. |
| **Vínculo de Perigo** | HAZ-01, HAZ-03 |
| **Status** | Implementado |

### REQ-35: Gate de Consentimento (LGPD Art. 11)

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-35 |
| **Descrição** | Como provedores de LLM atuam como subprocessadores de dados sensíveis de saúde (LGPD Art. 11), consentimento explícito e auditado é obrigatório antes de abrir o chat. |
| **Design** | `hasAuraPlusLlmConsent` consulta a tabela `user_consents` em busca de um grant explícito `aura_plus_llm`, verificando a trilha de auditoria (IP, User Agent, Timestamp). Falha fechado (fail-closed). |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/consent-gate.ts` |
| **Teste** | Bloqueio de acesso quando row de consentimento está ausente ou `granted` = false. |
| **Evidência** | Lógica de lookup na tabela `user_consents`. |
| **Vínculo de Perigo** | HAZ-05, HAZ-13 |
| **Status** | Implementado |

### REQ-36: Bloqueio Doctor-in-the-Loop do CDSS

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-36 |
| **Descrição** | O sistema atua como CDSS e não como prescritor. Qualquer protocolo clínico sugerido pelo LLM DEVE ser emitido como um `ProtocolBlock` com `"locked": true`. |
| **Design** | O System Prompt força estritamente o JSON de `protocolCard` a manter `locked: true`. O backend `checkAuraPlusGate` impede acesso a protocolos completos até que o utilizador tenha um registro de consulta (`unlocked_at != null`). |
| **Código-Fonte** | `gate.ts:checkAuraPlusGate`, `blocks.ts:ProtocolBlock`, `system-prompt.ts` |
| **Teste** | Assertir que o parser de output JSON descarta blocos de protocolo sem a flag locked. |
| **Evidência** | Tipagem `ProtocolBlock` em `blocks.ts`. |
| **Vínculo de Perigo** | HAZ-02, HAZ-04 |
| **Status** | Implementado |

### REQ-37: Output Streaming e Parser de Blocos

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-37 |
| **Descrição** | Respostas de IA devem ser parseadas de forma segura via um Tag Parser determinístico para isolar texto Markdown de blocos de UI funcionais (ex: protocolos), enviando-os sequencialmente via Server-Sent Events (SSE). |
| **Design** | `BlockTagParser` processa tokens brutos do LLM, matchando tags `<text>` e `<protocolCard>`, descartando alucinações fora das tags e enfileirando escritas SSE via `emitBlockEnd`. |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/chat.ts`, `aura-backend/src/health/aura-plus/sse.ts` |
| **Teste** | Testes de mock de parsing de stream. |
| **Evidência** | Frontend recebendo eventos `block_start` e `block_end` com segurança. |
| **Vínculo de Perigo** | HAZ-02, HAZ-04 |
| **Status** | Implementado |

### REQ-38: Âncoras Clínicas Medicine 3.0

| **Campo** | **Valor** |
|---|---|
| **ID do Requisito** | REQ-38 |
| **Descrição** | Todas as sugestões dentro da aplicação devem estar enraizadas em bases clínicas documentadas de medicina preventiva, evitando alegações diagnósticas sem suporte. |
| **Design** | `PROTOCOLS_CATALOG` define estruturas rígidas mapeando domínios, passos acionáveis, contraindicações e citações `evidenceAnchor` (ex: Attia, Lyon) limitando alucinação da IA. |
| **Código-Fonte** | `aura-backend/src/health/aura-plus/protocols-catalog.ts` |
| **Teste** | Verificação contra diretrizes do conselho médico consultivo. |
| **Evidência** | Campos `evidenceAnchor`. |
| **Vínculo de Perigo** | HAZ-02 |
| **Status** | Implementado |

---

## 4. Resumo de Cobertura

| **Categoria** | **Requisitos** | **Implementados** | **Planeados** | **Pendentes** |
|---|---|---|---|---|
| Avaliação Clínica (Avaliador Binário) | 8 | 8 | 0 | 0 |
| IA Conversacional e Segurança Clínica | 8 | 8 | 0 | 0 |
| Integração de Dados e Dispositivos | 4 | 4 | 0 | 0 |
| Cibersegurança e Privacidade | 13 | 13 | 0 | 0 |
| **Total** | **33** | **33** | **0** | **0** |

**Composição por Categoria:**

- **Avaliação Clínica (8):** REQ-01, REQ-02, REQ-12, REQ-13, REQ-14, REQ-24, REQ-25, REQ-29.
- **IA Conversacional (8):** REQ-03, REQ-04, REQ-21, REQ-33, REQ-34, REQ-36, REQ-37, REQ-38.
- **Integração de Dados (4):** REQ-10, REQ-15, REQ-28, REQ-32.
- **Cibersegurança (13):** REQ-05, REQ-06, REQ-07, REQ-08, REQ-09, REQ-11, REQ-16, REQ-17, REQ-18, REQ-19, REQ-20, REQ-31, REQ-35.

---

## 5. Índice de Referência Cruzada

### Por Perigo

| **ID do Perigo** | **Requisitos Relacionados** |
|---|---|
| HAZ-01 | REQ-01, REQ-02, REQ-12, REQ-13, REQ-15, REQ-25, REQ-29, REQ-34 |
| HAZ-02 | REQ-04, REQ-33, REQ-36, REQ-37, REQ-38 |
| HAZ-03 | REQ-04, REQ-34 |
| HAZ-04 | REQ-36, REQ-37 |
| HAZ-05 | REQ-03, REQ-19, REQ-35 |
| HAZ-06 | REQ-02, REQ-07, REQ-10 |
| HAZ-07 | REQ-05, REQ-06, REQ-08, REQ-09, REQ-31 |
| HAZ-08 | REQ-01, REQ-16, REQ-31 |
| HAZ-09 | REQ-24 |
| HAZ-10 | REQ-05, REQ-06, REQ-08, REQ-09, REQ-11, REQ-31 |
| HAZ-11 | REQ-10, REQ-13, REQ-14, REQ-28, REQ-29 |
| HAZ-12 | REQ-03, REQ-21 |
| HAZ-13 | REQ-17, REQ-18, REQ-35 |
| HAZ-14 | REQ-24 |
| HAZ-15 | REQ-09, REQ-11, REQ-16 |
| HAZ-16 | REQ-32 |
| HAZ-17 | REQ-15, REQ-28 |

### Por Requisito

| **ID do Requisito** | **Perigos Mitigados** |
|---|---|
| REQ-01 | HAZ-01, HAZ-08 |
| REQ-02 | HAZ-01, HAZ-06 |
| REQ-03 | HAZ-05, HAZ-12 |
| REQ-04 | HAZ-02, HAZ-03 |
| REQ-05 | HAZ-07, HAZ-10 |
| REQ-06 | HAZ-07, HAZ-10 |
| REQ-07 | HAZ-06 |
| REQ-08 | HAZ-07, HAZ-10 |
| REQ-09 | HAZ-07, HAZ-10, HAZ-15 |
| REQ-10 | HAZ-06, HAZ-11 |
| REQ-11 | HAZ-10, HAZ-15 |
| REQ-12 | HAZ-01 |
| REQ-13 | HAZ-01, HAZ-11 |
| REQ-14 | HAZ-11 |
| REQ-15 | HAZ-01, HAZ-17 |
| REQ-16 | HAZ-08, HAZ-15 |
| REQ-17 | HAZ-13 |
| REQ-18 | HAZ-13 |
| REQ-19 | HAZ-05 |
| REQ-20 | Nenhum (Regulatório) |
| REQ-21 | HAZ-12 |
| REQ-24 | HAZ-09, HAZ-14 |
| REQ-25 | HAZ-01 |
| REQ-28 | HAZ-11, HAZ-17 |
| REQ-29 | HAZ-01, HAZ-11 |
| REQ-31 | HAZ-07, HAZ-08, HAZ-10 |
| REQ-32 | HAZ-16 |
| REQ-33 | HAZ-02 |
| REQ-34 | HAZ-01, HAZ-03 |
| REQ-35 | HAZ-05, HAZ-13 |
| REQ-36 | HAZ-02, HAZ-04 |
| REQ-37 | HAZ-02, HAZ-04 |
| REQ-38 | HAZ-02 |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gestor de Qualidade | Frederico | | |
| Líder de Desenvolvimento | Arthur Teixeira de Almeida | | |
