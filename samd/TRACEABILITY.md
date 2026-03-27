# Requirements Traceability Matrix

**Document ID:** TM-001
**Revision:** 1.0
**Date:** 2026-03-27
**Product:** Aura Medical iOS Application
**Standard:** IEC 62304:2006+AMD1:2015 Section 5.1.1 (Software development planning)

---

## 1. Purpose

This document establishes bidirectional traceability between software requirements, design specifications, source code implementations, test procedures, and verification evidence. Each requirement is linked to its corresponding hazard(s) from RISK_ANALYSIS.md where applicable.

## 2. Conventions

- **Source paths** are relative to the repository root (`aura-ios/`).
- **Test status** indicates current verification state: Implemented, Planned, or Pending.
- **Hazard references** link to RISK_ANALYSIS.md identifiers.

---

## 3. Traceability Table

### REQ-01: Input Validation

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-01                                                                               |
| **Description**    | All numeric engine inputs must be validated against physiological ranges. Values outside bounds are rejected (set to nil) and logged. Glucose values > 600 mg/dL must be rejected. |
| **Design**         | `PhysiologicalRanges` struct in `EngineConfig` defines min/max for every biomarker. `validateInput()` checks each field and returns `ValidationResult` with rejected list. |
| **Source Code**    | `Engine/PhenomicEngine.swift:validateInput()` (lines 48--102), `Engine/EngineConfig.swift:PhysiologicalRanges` |
| **Test**           | Parity test vectors include boundary cases; unit tests for each biomarker range       |
| **Evidence**       | `phenomic-engine/parity-vectors.json` vector cases with out-of-range inputs           |
| **Hazard Link**    | HAZ-01, HAZ-08, HAZ-14                                                                |
| **Status**         | Implemented                                                                           |

### REQ-02: Score Range 0--100

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-02                                                                               |
| **Description**    | The Phenomic Engine must produce a `totalScore` integer in the range [0, 100]. Domain scores (slices) must each be in [0, 100]. |
| **Design**         | `PhenomicResult.totalScore` is computed as weighted average of domain scores, clamped to [0, 100]. Each slice score is independently bounded. |
| **Source Code**    | `Engine/PhenomicEngine.swift:compute()` -- final score clamping and rounding          |
| **Test**           | All 8 parity test vectors verify totalScore in [0, 100]; edge case vectors at boundaries |
| **Evidence**       | `phenomic-engine/parity-vectors.json`                                                 |
| **Hazard Link**    | HAZ-06, HAZ-17                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-03: SaMD Audit Trail

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-03                                                                               |
| **Description**    | Every engine computation must be logged to `ai_audit_log` with: user ID, input hash (SHA-256), output summary, engine version, processing time, confidence score, app version, and device ID. Logging must be best-effort and non-blocking. |
| **Design**         | `EngineAuditLogger` actor writes to Supabase `ai_audit_log` table. Input is hashed with SHA-256 (no raw PHI stored). Failures logged locally but do not block engine result. |
| **Source Code**    | `AuraMedical/Security/EngineAuditLogger.swift` -- `log()` method, `hashInput()`, `buildInputSummary()`, `buildOutputSummary()` |
| **Test**           | Integration test: verify audit row created after computation; verify non-blocking on network failure |
| **Evidence**       | `ai_audit_log` table in Supabase project `dhteseqmrgvhnuowgblp`                      |
| **Hazard Link**    | HAZ-12, HAZ-15                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-04: Cross-Validation Swift/TS

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-04                                                                               |
| **Description**    | Every iOS engine computation must be sent to the backend for recomputation by the canonical TypeScript engine. Score delta must be <= 2 points. Discrepancies must be logged. |
| **Design**         | `PhenomicService.sendBackendValidation()` fires detached Task to `AuraBackendClient.validateEngine()`. Backend endpoint `/api/engine/validate` recomputes and compares. |
| **Source Code**    | `AuraMedical/Services/PhenomicService.swift:sendBackendValidation()`, `AuraMedical/Services/AuraBackendClient.swift:validateEngine()` |
| **Test**           | Parity test vectors run on both engines; backend integration test for /api/engine/validate |
| **Evidence**       | Backend logs cross-validation results; `EngineValidationResponse` includes `delta` field |
| **Hazard Link**    | HAZ-04, HAZ-20                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-05: PHI Protection on Background

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-05                                                                               |
| **Description**    | All screens displaying PHI must apply a 30-radius Gaussian blur when the app enters background, is being screen-recorded, or is being screen-captured. |
| **Design**         | `PHIProtectionModifier` observes `scenePhase` and `UIScreen.capturedDidChangeNotification`. Applies blur when `scenePhase != .active` or `isCaptured == true`. |
| **Source Code**    | `AuraMedical/Security/PHIProtectionModifier.swift` -- `.phiProtected()` view modifier  |
| **Test**           | Manual QA: verify blur on app switch, screen recording, and screenshot                |
| **Evidence**       | Applied to BioHubView, DomainDetailView, PersonalDataView, ScoreView                 |
| **Hazard Link**    | HAZ-10                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-06: Biometric Authentication

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-06                                                                               |
| **Description**    | When enabled by the user, Face ID / Touch ID must be required to access the app. Falls back to device passcode. Authentication state stored in Keychain (not UserDefaults). |
| **Design**         | `BiometricAuthManager` (@Observable) uses `LAContext.evaluatePolicy(.deviceOwnerAuthentication)`. Enable/disable persisted in Keychain via `AuraKeychain`. |
| **Source Code**    | `AuraMedical/Security/BiometricAuthManager.swift` -- `authenticate()`, `isEnabled` property |
| **Test**           | Manual QA: enable biometric lock, verify lock screen appears on foreground            |
| **Evidence**       | Keychain entry `biometricAuth.enabled`                                                |
| **Hazard Link**    | HAZ-10                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-07: Certificate Pinning

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-07                                                                               |
| **Description**    | All connections to Supabase hosts must use SPKI SHA-256 certificate pinning with fail-closed policy. Backup pin for intermediate CA must be included for rotation resilience. |
| **Design**         | `PinnedURLSessionDelegate` extracts public key from server certificate, prepends correct ASN.1 header (EC P-256 or RSA), computes SHA-256, and compares against pinned hash set. Rejects connection if no match. |
| **Source Code**    | `AuraMedical/Security/CertificatePinning.swift` -- `PinnedURLSessionDelegate`, `URLSession.pinned` extension |
| **Test**           | Integration test: verify connection succeeds to valid hosts; verify rejection on pin mismatch |
| **Evidence**       | Pinned hosts: `dhteseqmrgvhnuowgblp.supabase.co`, `lldzbrrjamurobvticek.supabase.co` |
| **Hazard Link**    | HAZ-03, HAZ-09                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-08: Session Idle Timeout

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-08                                                                               |
| **Description**    | When biometric auth is enabled, the app must auto-lock after 30 minutes of idle time (no user interaction). |
| **Design**         | `BiometricAuthManager` tracks `lastInteractionDate` and runs `idleTimer` that checks elapsed time against `idleTimeoutInterval` (30 * 60 seconds). |
| **Source Code**    | `AuraMedical/Security/BiometricAuthManager.swift` -- `idleTimer`, `idleTimeoutInterval`, `lastInteractionDate` |
| **Test**           | Manual QA: enable biometric lock, wait 30 minutes, verify lock screen appears         |
| **Evidence**       | `BiometricAuthManager.idleTimeoutInterval = 30 * 60`                                  |
| **Hazard Link**    | HAZ-10                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-09: Keychain Storage

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-09                                                                               |
| **Description**    | All sensitive data (tokens, keys, auth flags) must be stored in iOS Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` accessibility. Data must not migrate to backups or new devices. |
| **Design**         | `AuraKeychain` enum provides typed save/load/delete API. All operations use `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`. `deleteAll()` method clears all app entries on logout. |
| **Source Code**    | `AuraMedical/Security/AuraKeychain.swift` -- `save()`, `load()`, `delete()`, `deleteAll()` |
| **Test**           | Unit test: save/load/delete cycle; verify accessibility attribute in query             |
| **Evidence**       | Keychain service identifier: `med.aura.medical`                                       |
| **Hazard Link**    | HAZ-10, HAZ-16                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-10: Offline-First Sync

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-10                                                                               |
| **Description**    | App must function offline using locally persisted data (SwiftData). Reads from local storage first, then refreshes from Supabase. Writes go to local immediately with `needsUpload` flag, flushed to server asynchronously. Server wins on conflict. |
| **Design**         | `SyncManager` (@MainActor, @Observable) coordinates SwiftData <-> Supabase sync. ModelContainer uses `NSFileProtectionComplete` for encryption at rest. |
| **Source Code**    | `AuraMedical/LocalData/SyncManager.swift` -- `sync()`, `makeContainer()`             |
| **Test**           | Integration test: verify local read without network; verify sync after reconnection   |
| **Evidence**       | SwiftData models: `LocalScore`, `LocalProfile`, `LocalBiometrics`                     |
| **Hazard Link**    | HAZ-07                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-11: App Attest Device Integrity

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-11                                                                               |
| **Description**    | App must use Apple App Attest to generate and store a device attestation key on first launch. Key ID stored in Keychain. Attestation object sent to backend for verification on API calls. |
| **Design**         | `AppAttestManager` actor uses `DCAppAttestService` to generate key. Key ID cached in memory and persisted in Keychain. `attest()` produces attestation for server-provided challenge hash. |
| **Source Code**    | `AuraMedical/Security/AppAttestManager.swift` -- `generateKeyIfNeeded()`, `attest()`, `assertKey()` |
| **Test**           | Device test: verify key generation on supported device; verify Keychain persistence    |
| **Evidence**       | Keychain entry `appAttest.keyId`; attestation header in backend requests               |
| **Hazard Link**    | HAZ-10, HAZ-16                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-12: Five-Domain Scoring

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-12                                                                               |
| **Description**    | Engine must compute individual scores (0--100) for exactly five health domains: metabolic, cardiovascular, sleep, nervous, and inflammatory. Each domain uses specific biomarkers with configurable weights. |
| **Design**         | `PhenomicEngine.compute()` calculates each domain score using `systemWeights` and `clusterInputWeights` from `EngineConfig`. Results returned as `PhenomicSlice` array within `PhenomicResult`. |
| **Source Code**    | `Engine/PhenomicEngine.swift:compute()` -- domain score calculation sections           |
| **Test**           | Parity test vectors verify all 5 domain scores; boundary tests for each domain        |
| **Evidence**       | `phenomic-engine/parity-vectors.json` -- 8 vectors with per-domain outputs             |
| **Hazard Link**    | HAZ-01, HAZ-06                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-13: Ghost Mode for Missing Data

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-13                                                                               |
| **Description**    | When entire data categories are missing (no wearable, no labs), the engine must apply conservative score caps to prevent artificially high scores. Caps are configurable per domain. |
| **Design**         | `GhostMode` config defines caps: `metabolicBmiCap`, `cardioActivityCap`, `inflammatoryMetabolicCaps` (severe/moderate), `nervousInflammatoryCap`. Applied during domain score finalization. |
| **Source Code**    | `Engine/EngineConfig.swift:GhostMode`, `Engine/PhenomicEngine.swift` -- ghost mode application sections |
| **Test**           | Parity test vector with minimal data verifies ghost mode caps applied                  |
| **Evidence**       | Config values in `EngineConfig.default.ghostMode`                                      |
| **Hazard Link**    | HAZ-01, HAZ-07                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-14: Systemic Drag Penalty

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-14                                                                               |
| **Description**    | When a biomarker exceeds a severe threshold in one domain, related domains must receive a configurable penalty ("systemic drag") to reflect physiological cross-system impact. |
| **Design**         | `SystemicDragRule` array in config: each rule specifies a symptom, threshold, and array of target domain penalties. Applied after individual domain scoring. |
| **Source Code**    | `Engine/EngineConfig.swift:SystemicDragRule`, `Engine/PhenomicEngine.swift` -- systemic drag application |
| **Test**           | Parity test vector with extreme biomarker verifies cross-domain penalty                |
| **Evidence**       | Config values in `EngineConfig.default.systemicDragRules`                              |
| **Hazard Link**    | HAZ-01                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-15: Inference Layer Estimation

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-15                                                                               |
| **Description**    | Missing symptoms must be estimated using evidence-based correlations. Real data always overrides inference. Inferred values must be discounted by confidence level. All inferences must be audited with source, coefficient, and reference. |
| **Design**         | `InferenceLayer.infer()` iterates correlation matrix from config. Propagation threshold (>= 3) prevents weak signals. Discount factors reduce inferred values. Audit trail captures every inference step. |
| **Source Code**    | `Engine/InferenceLayer.swift` -- `infer()`, `getConfidenceLevel()`, `AuditEntry` struct |
| **Test**           | Unit tests for inference with known correlation inputs; verify discount factor application |
| **Evidence**       | `EngineConfig.default.inferenceCorrelations`, `EngineConfig.default.inferenceDiscountFactors` |
| **Hazard Link**    | HAZ-01, HAZ-11                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-16: Data Encryption at Rest

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-16                                                                               |
| **Description**    | All locally persisted health data must be encrypted at rest using iOS file protection. SwiftData store must use `NSFileProtectionComplete` (encrypted when device locked). |
| **Design**         | `SyncManager.makeContainer()` creates `ModelContainer` with `NSFileProtectionComplete`. Keychain uses `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`. |
| **Source Code**    | `AuraMedical/LocalData/SyncManager.swift:makeContainer()` -- `ModelConfiguration` with file protection |
| **Test**           | Verify ModelContainer configuration; verify file protection attribute on SQLite store   |
| **Evidence**       | SwiftData store path with NSFileProtectionComplete attribute                           |
| **Hazard Link**    | HAZ-09, HAZ-16                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-17: Account Deletion

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-17                                                                               |
| **Description**    | Users must be able to delete their account. Deletion uses soft-delete with 30-day hold before permanent purge. Server-side RPC handles cascading deletion across all tables. |
| **Design**         | `delete_my_account()` Supabase RPC function performs soft-delete. Related table rows cascade. 30-day hold allows recovery from accidental deletion. |
| **Source Code**    | Supabase RPC `delete_my_account()`; triggered from Profile settings in app             |
| **Test**           | Integration test: call RPC, verify soft-delete flag set; verify cascading deletes      |
| **Evidence**       | Supabase function definition; LGPD/App Store compliance                                |
| **Hazard Link**    | HAZ-13                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-18: Logout Clears Local Data

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-18                                                                               |
| **Description**    | On sign-out, all locally cached PHI must be deleted: SwiftData models (LocalScore, LocalProfile, LocalBiometrics), Keychain entries, and in-memory state. |
| **Design**         | `AuthViewModel.signOut()` calls Supabase sign-out, then deletes all SwiftData models via `ModelContext.delete(model:)`, calls `AuraKeychain.deleteAll()`, and resets authentication state. |
| **Source Code**    | `AuraMedical/ViewModels/AuthViewModel.swift:signOut()` -- SwiftData deletion, Keychain clearing |
| **Test**           | Integration test: sign out, verify SwiftData empty, verify Keychain cleared            |
| **Evidence**       | Code review of signOut() method                                                        |
| **Hazard Link**    | HAZ-13                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-19: No PHI in Logs

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-19                                                                               |
| **Description**    | Application logs must not contain PHI. All logging must use `os.Logger` with `privacy: .private` annotations for any potentially sensitive values. `print()` must not be used for PHI-adjacent data. Backend uses Pino with PHI field redaction. |
| **Design**         | All Security/ files use `Logger(subsystem: "med.aura.medical", category: ...)`. Sensitive values annotated `privacy: .private`. Audit logger hashes inputs instead of logging raw values. |
| **Source Code**    | All files in `AuraMedical/Security/` -- `os.Logger` usage throughout                  |
| **Test**           | Code review: grep for `print(` in Security/ and Engine/ directories; verify os.Logger usage |
| **Evidence**       | Static analysis of logging patterns                                                    |
| **Hazard Link**    | HAZ-05                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-20: Privacy Usage Descriptions

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-20                                                                               |
| **Description**    | Info.plist must include usage descriptions for all sensitive permissions: HealthKit (`NSHealthShareUsageDescription`, `NSHealthUpdateUsageDescription`), Face ID (`NSFaceIDUsageDescription`), Camera (`NSCameraUsageDescription`), Photo Library (`NSPhotoLibraryUsageDescription`). |
| **Design**         | All usage descriptions provided in PT-BR. Descriptions explain why each permission is needed in user-friendly language. |
| **Source Code**    | `project.yml` -- `info.plist` section with all `NS*UsageDescription` keys              |
| **Test**           | Build verification: confirm all descriptions present in compiled Info.plist             |
| **Evidence**       | Xcode project settings; App Store review compliance                                    |
| **Hazard Link**    | None (regulatory compliance)                                                           |
| **Status**         | Implemented                                                                           |

### REQ-21: Engine Version Tracking

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-21                                                                               |
| **Description**    | Engine version must be tracked and included in all audit log entries and cross-validation requests. Format: `X.Y.Z-swift` for iOS, `X.Y.Z` for TypeScript. |
| **Design**         | `PhenomicEngine.engineVersion` static constant. Included in `EngineAuditLogger.log()` as `modelVersion` and in `AuraBackendClient.validateEngine()` as `engine_version`. |
| **Source Code**    | `Engine/PhenomicEngine.swift:engineVersion` (currently "4.2.0-swift")                  |
| **Test**           | Verify version string format; verify presence in audit log rows                        |
| **Evidence**       | `ai_audit_log.model_version` column                                                    |
| **Hazard Link**    | HAZ-04                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-22: Deterministic Engine Output

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-22                                                                               |
| **Description**    | The Phenomic Engine must be a pure function: same input + same config = same output, always. No random state, no external dependencies, no side effects. |
| **Design**         | `PhenomicEngine` is a struct with only static methods. `compute()` takes `EngineComputeRequest` and `EngineConfig` and returns `PhenomicResult`. No mutable state, no I/O, no randomness. |
| **Source Code**    | `Engine/PhenomicEngine.swift` -- all methods are `static`                              |
| **Test**           | Parity test: same input vector produces identical output on repeated runs               |
| **Evidence**       | Architecture review: struct with static methods, no instance state                     |
| **Hazard Link**    | HAZ-12, HAZ-17                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-23: Soft Normalization with Floor

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-23                                                                               |
| **Description**    | All biomarker normalization must use soft normalization with a guaranteed floor of 0.1 (never zero). Supports both "lower is better" and "higher is better" (inverted) modes. |
| **Design**         | `PhenomicEngine.softNormalize()` maps values to [0.1, 1.0] range. Default mode: lower is better. Inverted mode: higher is better (steps, HRV, HDL). |
| **Source Code**    | `Engine/PhenomicEngine.swift:softNormalize()` (lines 25--36)                          |
| **Test**           | Unit tests for boundary values, inversion, and floor guarantee                         |
| **Evidence**       | Parity with TypeScript `softNormalize()` function                                      |
| **Hazard Link**    | HAZ-20                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-24: Gender-Specific Reference Ranges

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-24                                                                               |
| **Description**    | Reference ranges for WHR, HDL, ferritin, and DHEA-S must be gender-specific. Male and female ranges defined separately in config. |
| **Design**         | `ReferenceRanges` struct contains `male` and `female` `GenderRanges`. `forGender()` method selects correct range set. |
| **Source Code**    | `Engine/EngineConfig.swift:ReferenceRanges` -- `forGender()` method                   |
| **Test**           | Parity test vectors include male and female cases; verify different ranges applied      |
| **Evidence**       | Config values in `EngineConfig.default.referenceRanges`                                |
| **Hazard Link**    | HAZ-14                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-25: Sleep Score Architecture

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-25                                                                               |
| **Description**    | Sleep domain score must be computed from sleep architecture data: duration, REM %, deep %, and efficiency. Each component uses piecewise scoring with configurable thresholds and weights. |
| **Design**         | `PhenomicEngine.calculateSleepScore()` computes weighted average of duration, REM, deep, and efficiency scores using `SleepThresholds` config. Returns nil if no sleep data. |
| **Source Code**    | `Engine/PhenomicEngine.swift:calculateSleepScore()` (lines 134--200+)                 |
| **Test**           | Parity test vectors with various sleep architectures                                   |
| **Evidence**       | Config values in `EngineConfig.default.sleepThresholds`                                |
| **Hazard Link**    | HAZ-07                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-26: Activity Vector Fallback

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-26                                                                               |
| **Description**    | Activity vector must attempt: (1) steps data, (2) resting heart rate, (3) configurable fallback score ("safe pessimism"). Each step only used if previous is unavailable. |
| **Design**         | `PhenomicEngine.calculateActivityVector()` uses cascading fallback: steps -> RHR -> fallbackScore from `ActivityThresholds` config. |
| **Source Code**    | `Engine/PhenomicEngine.swift:calculateActivityVector()` (lines 109--127)               |
| **Test**           | Unit tests: verify each fallback level with selective nil inputs                       |
| **Evidence**       | `EngineConfig.default.thresholds.activity.fallbackScore`                               |
| **Hazard Link**    | HAZ-07                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-27: Remote Config with Fallback

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-27                                                                               |
| **Description**    | Engine configuration must be fetched from Supabase `engine_config` table. Must fall back to hardcoded `.default` config on fetch failure. Cached config used for instant startup. |
| **Design**         | `EngineConfigManager` actor: `warmup()` loads cache then refreshes from network. `refreshConfig()` fetches active config. Falls back to `.default` on any failure. |
| **Source Code**    | `Engine/Config/EngineConfigManager.swift` -- `warmup()`, `refreshConfig()`, `loadCached()` |
| **Test**           | Integration test: verify fallback on network failure; verify cache persistence          |
| **Evidence**       | `engine_config` table in Supabase; `UserDefaults` cache key                            |
| **Hazard Link**    | HAZ-18                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-28: HealthKit Background Delivery

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-28                                                                               |
| **Description**    | App must register for HealthKit background delivery to receive wearable data updates when not in foreground. Daily summaries synced to Supabase. |
| **Design**         | `HealthKitSyncService` actor processes background delivery callbacks. Aggregates daily metrics (HRV, RHR, steps, sleep) and upserts to `healthkit_daily_summaries` table. |
| **Source Code**    | `AuraMedical/Services/HealthKitSyncService.swift:syncToday()`                          |
| **Test**           | Integration test: trigger background delivery handler; verify Supabase upsert          |
| **Evidence**       | `healthkit_daily_summaries` table; HealthKit background delivery entitlement            |
| **Hazard Link**    | HAZ-07, HAZ-19                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-29: Precision Indicator

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-29                                                                               |
| **Description**    | The app must display a precision/confidence indicator showing the proportion of real vs. inferred/missing data used in the score computation. Score is considered "estimated" when precision < 40%. |
| **Design**         | `PrecisionStatus` computed from data availability (wearable, questionnaire, biometrics, labs). `PhenomicResult.precision` field. `PhenomicService.isEstimated` computed property. |
| **Source Code**    | `AuraMedical/Services/PhenomicService.swift` -- `precisionStatus`, `isEstimated`       |
| **Test**           | Verify precision badge reflects data availability; verify isEstimated threshold          |
| **Evidence**       | UI displays precision badge on ScoreView                                               |
| **Hazard Link**    | HAZ-01, HAZ-11                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-30: Five-Tier Qualitative Scale

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-30                                                                               |
| **Description**    | Scores must be mapped to a five-tier qualitative scale for user-friendly display: critical (0--20), warning (21--40), fair (41--60), good (61--80), excellent (81--100). |
| **Design**         | Score tier computed from `totalScore` integer. UI displays corresponding label, color, and icon per tier. |
| **Source Code**    | Score tier logic in view layer; `AuraColors` provides tier-appropriate colors           |
| **Test**           | Unit test: verify tier boundaries; visual QA on all tier displays                      |
| **Evidence**       | UI screenshots for each tier                                                           |
| **Hazard Link**    | HAZ-06                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-31: Supabase JWT Authentication

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-31                                                                               |
| **Description**    | All authenticated API calls must include a valid Supabase JWT in the Authorization header. Supports email/password, Google Sign-In, and Apple Sign-In. |
| **Design**         | `AuthViewModel` manages Supabase session. `AuraBackendClient` attaches JWT from `supabase.auth.session` to every request. |
| **Source Code**    | `AuraMedical/ViewModels/AuthViewModel.swift` -- `signIn()`, `signUp()`, `signInWithGoogle()`, `signInWithApple()`; `AuraMedical/Services/AuraBackendClient.swift:attachAuthHeader()` |
| **Test**           | Integration test: verify JWT present in backend requests; verify 401 on expired token  |
| **Evidence**       | Backend auth middleware validates JWT                                                   |
| **Hazard Link**    | HAZ-10                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-32: Score Persistence

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-32                                                                               |
| **Description**    | Every valid engine computation result must be persisted to both local storage (SwiftData LocalScore) and remote storage (Supabase daily_phenomic_scores). Includes audit log ID linkage. |
| **Design**         | `ScorePersistence.shared.persistIfNeeded()` writes to both stores. `daily_phenomic_scores.audit_log_id` links score to audit trail. |
| **Source Code**    | `Engine/ScorePersistence.swift:persistIfNeeded()`, `AuraMedical/LocalData/LocalScore.swift` |
| **Test**           | Integration test: compute score, verify LocalScore and daily_phenomic_scores rows       |
| **Evidence**       | SwiftData LocalScore model; Supabase `daily_phenomic_scores` table                     |
| **Hazard Link**    | HAZ-06, HAZ-12                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-33: Backend Client Certificate Pinning

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-33                                                                               |
| **Description**    | The AuraBackendClient must use the certificate-pinned URLSession for all backend API calls. Base URL configured via xcconfig (not hardcoded). |
| **Design**         | `AuraBackendClient` uses `URLSession.pinned` (shared pinned session). Base URL read from `Bundle.main.infoDictionary["AURA_BACKEND_URL"]`. Crashes on startup if not configured. |
| **Source Code**    | `AuraMedical/Services/AuraBackendClient.swift` -- `session: .pinned`, `baseURL` from xcconfig |
| **Test**           | Build verification: confirm xcconfig contains AURA_BACKEND_URL; verify .pinned session used |
| **Evidence**       | `Debug.xcconfig` / `Release.xcconfig` contain AURA_BACKEND_URL                         |
| **Hazard Link**    | HAZ-03, HAZ-18                                                                        |
| **Status**         | Implemented                                                                           |

### REQ-34: Data Cap Safety Limits

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-34                                                                               |
| **Description**    | Domain scores must be capped when critical biomarkers exceed safety thresholds. Prevents artificially high scores when dangerous values are present. Configurable per domain per biomarker. |
| **Design**         | `DataCaps` config defines per-biomarker threshold/cap pairs for cardio (BP), metabolic (HbA1c, HOMA-IR, BMI), and inflammatory (hs-CRP). Applied after domain score calculation. |
| **Source Code**    | `Engine/EngineConfig.swift:DataCaps`, `Engine/PhenomicEngine.swift` -- data cap application |
| **Test**           | Parity test vectors with extreme biomarker values verify cap application                |
| **Evidence**       | Config values in `EngineConfig.default.dataCaps`                                       |
| **Hazard Link**    | HAZ-01                                                                                |
| **Status**         | Implemented                                                                           |

### REQ-35: Confidence Level Computation

| Field              | Value                                                                                |
|--------------------|--------------------------------------------------------------------------------------|
| **Requirement ID** | REQ-35                                                                               |
| **Description**    | Confidence level (1--5) computed from data availability: 1 (questionnaire only), 2 (biometrics), 3 (wearables), 4 (labs), 5 (genetics). Used by Inference Layer for discount factor selection. |
| **Design**         | `InferenceLayer.getConfidenceLevel()` evaluates data flags in priority order. Higher confidence = higher discount factor = more trusted inferences. |
| **Source Code**    | `Engine/InferenceLayer.swift:getConfidenceLevel()` (lines 34--45)                      |
| **Test**           | Unit tests for each confidence level tier with appropriate data flags                   |
| **Evidence**       | `EngineConfig.default.inferenceDiscountFactors` mapping                                 |
| **Hazard Link**    | HAZ-11                                                                                |
| **Status**         | Implemented                                                                           |

---

## 4. Coverage Summary

| Category              | Requirements | Implemented | Planned | Pending |
|-----------------------|-------------|-------------|---------|---------|
| Engine Core           | 12          | 12          | 0       | 0       |
| Security              | 10          | 10          | 0       | 0       |
| Data Management       | 7           | 7           | 0       | 0       |
| Infrastructure        | 6           | 6           | 0       | 0       |
| **Total**             | **35**      | **35**      | **0**   | **0**   |

## 5. Cross-Reference Index

### By Hazard

| Hazard ID | Related Requirements                    |
|-----------|-----------------------------------------|
| HAZ-01    | REQ-01, REQ-12, REQ-13, REQ-14, REQ-15, REQ-29, REQ-34 |
| HAZ-02    | (Mitigated by disclaimers -- see INTENDED_USE.md) |
| HAZ-03    | REQ-07, REQ-33                          |
| HAZ-04    | REQ-04, REQ-21                          |
| HAZ-05    | REQ-19                                  |
| HAZ-06    | REQ-02, REQ-12, REQ-30, REQ-32         |
| HAZ-07    | REQ-10, REQ-13, REQ-25, REQ-26, REQ-28 |
| HAZ-08    | REQ-01                                  |
| HAZ-09    | REQ-07, REQ-16                          |
| HAZ-10    | REQ-05, REQ-06, REQ-08, REQ-09, REQ-11, REQ-31 |
| HAZ-11    | REQ-15, REQ-29, REQ-35                  |
| HAZ-12    | REQ-03, REQ-22, REQ-32                  |
| HAZ-13    | REQ-17, REQ-18                          |
| HAZ-14    | REQ-01, REQ-24                          |
| HAZ-15    | REQ-03                                  |
| HAZ-16    | REQ-09, REQ-11, REQ-16                  |
| HAZ-17    | REQ-02, REQ-22                          |
| HAZ-18    | REQ-27, REQ-33                          |
| HAZ-19    | REQ-13, REQ-28                          |
| HAZ-20    | REQ-04, REQ-23                          |

### By Source File

| Source File                                      | Requirements                          |
|--------------------------------------------------|---------------------------------------|
| `Engine/PhenomicEngine.swift`                    | REQ-01, REQ-02, REQ-12, REQ-13, REQ-14, REQ-22, REQ-23, REQ-25, REQ-26, REQ-34 |
| `Engine/EngineConfig.swift`                      | REQ-01, REQ-13, REQ-14, REQ-24, REQ-34 |
| `Engine/InferenceLayer.swift`                    | REQ-15, REQ-35                        |
| `Engine/ScorePersistence.swift`                  | REQ-32                                |
| `Engine/Config/EngineConfigManager.swift`        | REQ-27                                |
| `AuraMedical/Security/EngineAuditLogger.swift`   | REQ-03, REQ-21                        |
| `AuraMedical/Security/PHIProtectionModifier.swift` | REQ-05                              |
| `AuraMedical/Security/BiometricAuthManager.swift` | REQ-06, REQ-08                       |
| `AuraMedical/Security/CertificatePinning.swift`  | REQ-07, REQ-33                        |
| `AuraMedical/Security/AuraKeychain.swift`        | REQ-09                                |
| `AuraMedical/Security/AppAttestManager.swift`    | REQ-11                                |
| `AuraMedical/LocalData/SyncManager.swift`        | REQ-10, REQ-16                        |
| `AuraMedical/Services/PhenomicService.swift`     | REQ-04, REQ-29                        |
| `AuraMedical/Services/AuraBackendClient.swift`   | REQ-04, REQ-33                        |
| `AuraMedical/Services/HealthKitSyncService.swift` | REQ-28                               |
| `AuraMedical/ViewModels/AuthViewModel.swift`     | REQ-18, REQ-31                        |

---

**Approval Signatures:**

| Role                    | Name | Date | Signature |
|-------------------------|------|------|-----------|
| Quality Manager         |      |      |           |
| Software Development Lead |    |      |           |
