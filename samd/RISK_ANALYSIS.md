# Risk Analysis (ISO 14971 FMEA)

**Document ID:** RA-001
**Revision:** 1.0
**Date:** 2026-03-27
**Product:** Aura Medical iOS Application
**Standard:** ISO 14971:2019 -- Application of risk management to medical devices
**Method:** Failure Mode and Effects Analysis (FMEA)
**Classification:** SaMD Class B (IEC 62304), IMDRF Risk Category II

---

## 1. Scope

This risk analysis covers all identified hazards associated with the Aura Medical iOS application, including software failures, data integrity issues, security threats, and use errors. The analysis follows the FMEA methodology with Risk Priority Numbers (RPN) calculated as:

**RPN = Severity x Likelihood x Detection**

Where each factor is rated on a scale of 1--5:

### Severity Scale

| Rating | Level      | Description                                                                |
|--------|------------|----------------------------------------------------------------------------|
| 1      | Negligible | No impact on user health decisions; cosmetic issue only                    |
| 2      | Minor      | Slight inconvenience; user may notice but no behavioral impact             |
| 3      | Moderate   | User may make a suboptimal wellness decision based on incorrect information |
| 4      | Major      | User may delay seeking medical care or make a harmful lifestyle change      |
| 5      | Critical   | User may forgo necessary medical treatment based on false reassurance       |

### Likelihood Scale

| Rating | Level    | Description                                  |
|--------|----------|----------------------------------------------|
| 1      | Rare     | Less than once per 100,000 uses              |
| 2      | Unlikely | Once per 10,000--100,000 uses                |
| 3      | Possible | Once per 1,000--10,000 uses                  |
| 4      | Likely   | Once per 100--1,000 uses                     |
| 5      | Frequent | More than once per 100 uses                  |

### Detection Scale

| Rating | Level          | Description                                                     |
|--------|----------------|-----------------------------------------------------------------|
| 1      | Almost certain | Automated detection with blocking action (e.g., input validation rejects) |
| 2      | High           | Automated detection with alert/log (e.g., cross-validation delta) |
| 3      | Moderate       | Detectable through routine testing or monitoring                 |
| 4      | Low            | Detectable only through targeted investigation                   |
| 5      | Undetectable   | No detection mechanism currently in place                        |

### RPN Thresholds

| RPN Range | Risk Level  | Action Required                                       |
|-----------|-------------|-------------------------------------------------------|
| 1--15     | Acceptable  | No additional action required; monitor                 |
| 16--39    | Low         | Document mitigation; review at next design cycle       |
| 40--74    | Medium      | Implement additional controls before release           |
| 75--125   | High        | Requires immediate mitigation; management review       |

---

## 2. Hazard Analysis Table

### HAZ-01: Falsely High Score for Clinically Unwell Patient

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-01                                                                              |
| **Hazard**      | Engine produces a high (good) score for a patient who is clinically unwell           |
| **Cause**       | Missing lab data; inference layer fills gaps with optimistic estimates; user has not entered biomarkers |
| **Effect**      | User falsely reassured; may delay seeking necessary medical care                     |
| **Severity**    | 5                                                                                   |
| **Likelihood**  | 3                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **30**                                                                               |
| **Mitigation**  | (1) Input validation rejects physiologically implausible values (REQ-01). (2) Ghost Mode applies conservative caps when data is missing (REQ-13). (3) Inference Layer uses discount factors that reduce confidence of estimated values (REQ-15). (4) Precision indicator clearly shows data completeness percentage. (5) All screens display disclaimer: "This is not a medical diagnosis." (6) Systemic Drag penalty cross-penalizes domains when severe markers detected (REQ-14). |
| **Residual Risk** | Low. Multiple conservative safeguards plus mandatory disclaimer reduce risk of false reassurance. Score is explicitly labeled as wellness estimate, not diagnostic. |
| **Traceability** | REQ-01, REQ-13, REQ-14, REQ-15; INTENDED_USE.md Section 2.2 |

### HAZ-02: User Acts on Score Without Medical Guidance

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-02                                                                              |
| **Hazard**      | User makes health decisions (diet, exercise, supplements) based solely on score without consulting a physician |
| **Cause**       | User interprets wellness score as medical advice; insufficient disclaimers           |
| **Effect**      | Inappropriate self-treatment; delay in professional diagnosis                        |
| **Severity**    | 4                                                                                   |
| **Likelihood**  | 4                                                                                   |
| **Detection**   | 4                                                                                   |
| **RPN**         | **64**                                                                               |
| **Mitigation**  | (1) Prominent disclaimers on ScoreView, DomainDetailView, and ResultsView. (2) Intended use statement explicitly excludes diagnostic claims. (3) App Store description includes wellness-only positioning. (4) In-app text uses "wellness estimate" language, never "diagnosis" or "treatment." (5) Onboarding flow includes disclaimer acceptance. |
| **Residual Risk** | Medium. User behavior cannot be fully controlled, but comprehensive disclaimers and non-diagnostic language reduce likelihood of misinterpretation. Residual risk accepted per benefit-risk analysis. |
| **Traceability** | INTENDED_USE.md Sections 2.2, 7 |

### HAZ-03: Biometric Data Corrupted in Transit

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-03                                                                              |
| **Hazard**      | Health data (lab results, biometrics) corrupted or intercepted during network transmission |
| **Cause**       | Man-in-the-middle attack; TLS downgrade; compromised certificate authority           |
| **Effect**      | Corrupted data produces incorrect score; PHI exposed to attacker                     |
| **Severity**    | 4                                                                                   |
| **Likelihood**  | 1                                                                                   |
| **Detection**   | 1                                                                                   |
| **RPN**         | **4**                                                                                |
| **Mitigation**  | (1) TLS 1.2+ enforced for all network connections. (2) Certificate pinning via SPKI SHA-256 hashes with fail-closed policy (REQ-07). (3) Backup pin (intermediate CA) for certificate rotation. (4) ASN.1 header validation for EC P-256 and RSA key types. (5) Pinned hosts explicitly enumerated; connections to unknown hosts rejected. |
| **Residual Risk** | Very low. Certificate pinning with fail-closed design and backup pins makes MITM attacks practically infeasible. |
| **Traceability** | REQ-07; `Security/CertificatePinning.swift` |

### HAZ-04: Engine Divergence Between Swift and TypeScript

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-04                                                                              |
| **Hazard**      | Swift engine produces different score than canonical TypeScript engine for identical inputs |
| **Cause**       | Code drift after update to one engine without propagating to the other; floating-point differences; config sync failure |
| **Effect**      | Inconsistent scores across platforms; audit trail discrepancies; loss of regulatory traceability |
| **Severity**    | 3                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 1                                                                                   |
| **RPN**         | **6**                                                                                |
| **Mitigation**  | (1) Cross-validation: every iOS computation is sent to backend for TS engine recomputation; delta must be <= 2 points (REQ-04). (2) Parity test vectors (8 canonical cases) validated on both engines. (3) 3-repo sync protocol documented in CONFIG_MGMT.md. (4) Engine version stamped in audit log for traceability. (5) PR review required for any Engine/ directory changes. |
| **Residual Risk** | Very low. Automated cross-validation on every computation catches drift immediately. Parity tests prevent regression. |
| **Traceability** | REQ-04; CONFIG_MGMT.md; `Services/PhenomicService.swift:sendBackendValidation()` |

### HAZ-05: PHI Exposed in Application Logs

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-05                                                                              |
| **Hazard**      | Protected Health Information (PHI) written to console logs, crash reports, or analytics |
| **Cause**       | Developer uses `print()` instead of `os.Logger`; log message includes raw biomarker values |
| **Effect**      | PHI exposure in device logs accessible via Xcode or crash reporting services          |
| **Severity**    | 4                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **16**                                                                               |
| **Mitigation**  | (1) All Security/ files use `os.Logger` with `privacy: .private` annotations (REQ-19). (2) Backend uses Pino logger with PHI field redaction. (3) Code review checklist includes PHI logging verification. (4) Audit logger hashes inputs (SHA-256) instead of storing raw values. (5) No PHI in crash reports (only anonymized error descriptions). |
| **Residual Risk** | Low. Systematic use of privacy-aware logging and input hashing prevents PHI leakage. Code review process provides additional catch. |
| **Traceability** | REQ-19; `Security/EngineAuditLogger.swift:hashInput()` |

### HAZ-06: Score Display Error in UI

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-06                                                                              |
| **Hazard**      | UI displays incorrect score due to binding error, race condition, or rendering bug    |
| **Cause**       | SwiftUI @Published state not properly propagated; concurrent update; stale cache      |
| **Effect**      | User sees wrong score, potentially making incorrect wellness assessment               |
| **Severity**    | 3                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 3                                                                                   |
| **RPN**         | **18**                                                                               |
| **Mitigation**  | (1) PhenomicService is the single source of truth for all score displays. (2) @Published property wrapper ensures SwiftUI reactivity. (3) Score persisted to SwiftData (LocalScore) and Supabase for verification. (4) Manual QA testing on score display screens. (5) Engine result is immutable struct — no mutation after computation. |
| **Residual Risk** | Low. Single-source-of-truth architecture and immutable result structs minimize display errors. |
| **Traceability** | REQ-02; `Services/PhenomicService.swift`; `ViewModels/DashboardViewModel.swift` |

### HAZ-07: HealthKit Data Stale or Missing

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-07                                                                              |
| **Hazard**      | Wearable data from HealthKit is stale (days old) or completely absent                |
| **Cause**       | User removes wearable; HealthKit authorization revoked; Apple Watch not worn; background delivery fails |
| **Effect**      | Score computed with outdated or missing wearable inputs, reducing accuracy            |
| **Severity**    | 2                                                                                   |
| **Likelihood**  | 4                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **16**                                                                               |
| **Mitigation**  | (1) Ghost Mode applies conservative caps when wearable data is absent (REQ-13). (2) Activity vector falls back to configurable pessimistic score. (3) Precision indicator shows reduced confidence without wearable data. (4) HealthKitSyncService uses background delivery for timely updates. (5) Inference Layer can estimate wearable-dependent symptoms from lab data with discount factors. |
| **Residual Risk** | Low. Conservative fallbacks ensure score never overstates wellness when data is missing. Precision badge clearly communicates data completeness. |
| **Traceability** | REQ-13, REQ-15; `Engine/EngineConfig.swift:GhostMode`; `Services/HealthKitSyncService.swift` |

### HAZ-08: Incorrect Lab OCR Extraction

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-08                                                                              |
| **Hazard**      | OCR misreads lab result values from uploaded document (e.g., "1.5" read as "15")     |
| **Cause**       | Poor image quality; unusual lab report format; OCR model error                       |
| **Effect**      | Grossly incorrect biomarker value enters engine, producing misleading score           |
| **Severity**    | 4                                                                                   |
| **Likelihood**  | 3                                                                                   |
| **Detection**   | 1                                                                                   |
| **RPN**         | **12**                                                                               |
| **Mitigation**  | (1) Input validation rejects values outside physiological ranges (REQ-01) — e.g., glucose > 600 is rejected. (2) User review screen shows extracted values for manual confirmation before saving. (3) PhysiologicalRanges config defines min/max for every lab biomarker. (4) Rejected values logged with reason for audit trail. |
| **Residual Risk** | Very low. Physiological range validation catches order-of-magnitude OCR errors. User review provides final checkpoint. |
| **Traceability** | REQ-01; `Engine/PhenomicEngine.swift:validateInput()`; `Services/LabOCRService.swift` |

### HAZ-09: Database Breach Exposing PHI

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-09                                                                              |
| **Hazard**      | Unauthorized access to Supabase database exposes user health records                 |
| **Cause**       | SQL injection; misconfigured RLS policies; compromised service key; insider threat    |
| **Effect**      | Mass PHI exposure; regulatory violation (LGPD, HIPAA)                                |
| **Severity**    | 5                                                                                   |
| **Likelihood**  | 1                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **10**                                                                               |
| **Mitigation**  | (1) Supabase Row-Level Security (RLS) on all tables containing PHI. (2) Encryption at rest (Supabase default AES-256). (3) Certificate pinning prevents MITM to database (REQ-07). (4) pgaudit extension for database access logging. (5) Service role key never exposed to client; only anon key used from iOS. (6) Parameterized queries via Supabase client SDK (no raw SQL from client). |
| **Residual Risk** | Very low. Defense-in-depth with RLS, encryption, audit logging, and pinning provides comprehensive protection. |
| **Traceability** | REQ-07, REQ-16; `Services/SupabaseClient.swift` |

### HAZ-10: Session Hijacking

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-10                                                                              |
| **Hazard**      | Attacker gains access to user's authenticated session                                |
| **Cause**       | Stolen JWT token; shared device without screen lock; background session theft         |
| **Effect**      | Unauthorized access to user's health data and account                                |
| **Severity**    | 4                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **16**                                                                               |
| **Mitigation**  | (1) Biometric authentication required when enabled (REQ-06). (2) 30-minute idle timeout auto-locks app (REQ-08). (3) PHI blur on background (REQ-05) prevents screenshot capture of health data. (4) JWT tokens stored in Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` (REQ-09). (5) App Attest validates device integrity (REQ-11). |
| **Residual Risk** | Low. Multi-layered authentication and session management significantly reduce hijacking risk. |
| **Traceability** | REQ-05, REQ-06, REQ-08, REQ-09, REQ-11 |

### HAZ-11: Inference Layer Produces Misleading Estimate

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-11                                                                              |
| **Hazard**      | Inference Layer estimates missing symptoms inaccurately, distorting domain scores     |
| **Cause**       | Weak correlation used for inference; source value near propagation threshold; multiple chained inferences |
| **Effect**      | Domain score inflated or deflated based on unreliable estimates                       |
| **Severity**    | 3                                                                                   |
| **Likelihood**  | 3                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **18**                                                                               |
| **Mitigation**  | (1) Discount factors reduce inferred value confidence based on data tier (REQ-15). (2) Propagation threshold (>= 3) prevents weak signals from triggering inference. (3) Real data always overrides inferred values. (4) Audit trail logs all inferred values with source, coefficient, and reference. (5) Ghost Mode default value (3) serves as conservative baseline. (6) Precision indicator reflects inference proportion. |
| **Residual Risk** | Low. Conservative discount factors, propagation threshold, and real-data override prevent meaningful distortion. |
| **Traceability** | REQ-15; `Engine/InferenceLayer.swift` |

### HAZ-12: Audit Trail Write Failure

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-12                                                                              |
| **Hazard**      | Engine audit log entry fails to write to Supabase                                    |
| **Cause**       | Network connectivity loss; Supabase service outage; authentication expiry             |
| **Effect**      | Gap in SaMD audit trail; computation not traceable for regulatory review              |
| **Severity**    | 2                                                                                   |
| **Likelihood**  | 3                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **12**                                                                               |
| **Mitigation**  | (1) Audit logging is best-effort and non-blocking — engine result is still valid (REQ-03). (2) Failure logged via os.Logger for local device diagnostics. (3) Engine is deterministic: computation can be replayed from input hash + engine version. (4) Score persisted to SwiftData (LocalScore) with audit_log_id if available. (5) daily_phenomic_scores table provides secondary audit record. |
| **Residual Risk** | Low. Deterministic engine enables replay-based auditing even when primary audit write fails. Non-blocking design ensures user experience is unaffected. |
| **Traceability** | REQ-03; `Security/EngineAuditLogger.swift` |

### HAZ-13: Incomplete Account Deletion

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-13                                                                              |
| **Hazard**      | User requests account deletion but data persists in database or local storage         |
| **Cause**       | Deletion RPC fails silently; local SwiftData not cleared; orphaned records in related tables |
| **Effect**      | Regulatory violation (LGPD right to erasure); user trust breach                       |
| **Severity**    | 3                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 3                                                                                   |
| **RPN**         | **18**                                                                               |
| **Mitigation**  | (1) Server-side `delete_my_account()` RPC performs soft-delete with 30-day retention hold (REQ-17). (2) Local SwiftData cleared on signOut: LocalScore, LocalProfile, LocalBiometrics deleted (REQ-18). (3) Keychain cleared via `AuraKeychain.deleteAll()`. (4) 30-day hold allows recovery from accidental deletion before permanent purge. (5) Cascading deletes configured in Supabase for related tables. |
| **Residual Risk** | Low. Comprehensive deletion across all storage layers with safety hold period balances erasure completeness with recovery capability. |
| **Traceability** | REQ-17, REQ-18; `ViewModels/AuthViewModel.swift:signOut()` |

### HAZ-14: Wrong Age or Gender Input

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-14                                                                              |
| **Hazard**      | User enters incorrect age or gender, affecting reference range selection              |
| **Cause**       | Typo in age; user selects wrong gender; intentional misentry                         |
| **Effect**      | Gender-specific reference ranges (WHR, HDL, ferritin, DHEA-S) applied incorrectly; score distorted |
| **Severity**    | 3                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 3                                                                                   |
| **RPN**         | **18**                                                                               |
| **Mitigation**  | (1) Age validated against physiological range [18--120] in validateInput(); out-of-range clamped to bounds. (2) Questionnaire UI uses picker controls (not free text) for gender selection. (3) Profile screen allows users to review and correct demographics. (4) Gender-specific ranges use gradual normalization (not binary thresholds), reducing impact of misentry. |
| **Residual Risk** | Low. Input validation and clamping prevent extreme values; gradual normalization reduces sensitivity to minor errors. |
| **Traceability** | REQ-01; `Engine/PhenomicEngine.swift:validateInput()` |

### HAZ-15: Device Clock Manipulation

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-15                                                                              |
| **Hazard**      | User or malware manipulates device clock, corrupting temporal data                   |
| **Cause**       | Manual clock change; timezone confusion; jailbroken device                           |
| **Effect**      | Score timestamps incorrect; HealthKit data assigned to wrong dates; sync conflicts    |
| **Severity**    | 2                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 3                                                                                   |
| **RPN**         | **12**                                                                               |
| **Mitigation**  | (1) Supabase uses server-side `now()` timestamps for all database writes. (2) Audit log timestamps are server-generated, not client-supplied. (3) HealthKit provides its own timestamps from Apple's time service. (4) SyncManager uses server timestamps for conflict resolution. |
| **Residual Risk** | Very low. Server-side timestamps for all critical records eliminate client clock dependency. |
| **Traceability** | REQ-03; `Security/EngineAuditLogger.swift`; `LocalData/SyncManager.swift` |

### HAZ-16: Jailbroken Device Bypass

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-16                                                                              |
| **Hazard**      | User runs app on jailbroken device, bypassing iOS security controls                  |
| **Cause**       | Jailbroken iOS removes sandbox restrictions; Keychain may be accessible               |
| **Effect**      | PHI exposure through Keychain extraction; biometric auth bypass                       |
| **Severity**    | 4                                                                                   |
| **Likelihood**  | 1                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **8**                                                                                |
| **Mitigation**  | (1) App Attest validates device integrity (REQ-11). (2) Keychain uses `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` — data does not migrate to backups (REQ-09). (3) NSFileProtectionComplete encrypts SwiftData store when device locked (REQ-16). (4) Certificate pinning operates independently of device integrity. |
| **Residual Risk** | Low. App Attest provides attestation signal; Keychain and file protection provide defense even on compromised devices. |
| **Traceability** | REQ-09, REQ-11, REQ-16; `Security/AppAttestManager.swift` |

### HAZ-17: Concurrent Engine Computation Race Condition

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-17                                                                              |
| **Hazard**      | Multiple simultaneous engine computations produce conflicting results                 |
| **Cause**       | User triggers recompute while previous computation is in progress; background refresh overlaps manual refresh |
| **Effect**      | Race condition may display intermediate or stale result                               |
| **Severity**    | 2                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 3                                                                                   |
| **RPN**         | **12**                                                                               |
| **Mitigation**  | (1) PhenomicEngine.compute() is a pure static function with no mutable state. (2) PhenomicService is @MainActor, serializing state updates. (3) isLoading flag prevents UI-triggered concurrent computations. (4) Engine result is an immutable struct assigned atomically to @Published property. |
| **Residual Risk** | Very low. Stateless engine and MainActor serialization eliminate race conditions by design. |
| **Traceability** | REQ-02; `Engine/PhenomicEngine.swift`; `Services/PhenomicService.swift` |

### HAZ-18: Remote Config Poisoning

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-18                                                                              |
| **Hazard**      | Attacker modifies remote engine configuration to alter scoring behavior               |
| **Cause**       | Compromised Supabase credentials; malicious config entry in engine_config table       |
| **Effect**      | Engine produces systematically biased scores for all users                            |
| **Severity**    | 5                                                                                   |
| **Likelihood**  | 1                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **10**                                                                               |
| **Mitigation**  | (1) Engine config fetched via RLS-protected table (only service role can write). (2) EngineConfigManager falls back to hardcoded .default config on fetch failure. (3) Config cached in UserDefaults; poisoned config replaced on next valid fetch. (4) Cross-validation catches score divergence caused by config mismatch. (5) Certificate pinning prevents MITM config injection. |
| **Residual Risk** | Very low. RLS write protection and hardcoded fallback config provide defense-in-depth. |
| **Traceability** | `Engine/Config/EngineConfigManager.swift`; REQ-07 |

### HAZ-19: HealthKit Authorization Silently Revoked

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-19                                                                              |
| **Hazard**      | User revokes HealthKit permissions in Settings without notifying the app              |
| **Cause**       | iOS allows permission revocation at any time without app callback                    |
| **Effect**      | App attempts HealthKit reads that silently return no data; score degrades without explanation |
| **Severity**    | 2                                                                                   |
| **Likelihood**  | 3                                                                                   |
| **Detection**   | 2                                                                                   |
| **RPN**         | **12**                                                                               |
| **Mitigation**  | (1) HealthKitManager checks authorization status before each read. (2) Ghost Mode activates automatically when wearable data is nil. (3) Precision indicator drops to reflect missing wearable data. (4) Wearables connection screen shows current authorization state. |
| **Residual Risk** | Low. Graceful degradation via Ghost Mode ensures scores remain conservative. |
| **Traceability** | REQ-13; `Services/HealthKitManager.swift` |

### HAZ-20: Floating-Point Precision Loss in Score Calculation

| Field           | Value                                                                               |
|-----------------|-------------------------------------------------------------------------------------|
| **Hazard ID**   | HAZ-20                                                                              |
| **Hazard**      | Floating-point arithmetic differences between Swift Double and TypeScript Number produce score divergence |
| **Cause**       | IEEE 754 rounding differences; different compiler optimizations; intermediate precision loss |
| **Effect**      | Cross-validation delta exceeds threshold; audit trail inconsistency                   |
| **Severity**    | 2                                                                                   |
| **Likelihood**  | 2                                                                                   |
| **Detection**   | 1                                                                                   |
| **RPN**         | **4**                                                                                |
| **Mitigation**  | (1) Cross-validation tolerance of delta <= 2 accounts for floating-point differences. (2) Both engines use IEEE 754 64-bit doubles. (3) Final score rounded to integer (0--100), eliminating sub-integer precision differences. (4) Parity test vectors validate exact-match on integer outputs. |
| **Residual Risk** | Very low. Integer rounding of final score eliminates floating-point precision as a practical concern. |
| **Traceability** | REQ-04; `Engine/PhenomicEngine.swift:compute()` |

---

## 3. Risk Summary Matrix

| RPN Range     | Count | Hazard IDs                                               |
|---------------|-------|----------------------------------------------------------|
| 1--15 (Acceptable)  | 10    | HAZ-03, HAZ-04, HAZ-08, HAZ-09, HAZ-12, HAZ-15, HAZ-16, HAZ-17, HAZ-18, HAZ-20 |
| 16--39 (Low)        | 8     | HAZ-01, HAZ-05, HAZ-06, HAZ-07, HAZ-10, HAZ-11, HAZ-13, HAZ-14 |
| 40--74 (Medium)     | 2     | HAZ-02, HAZ-19                                            |
| 75--125 (High)      | 0     | None                                                      |

**Overall Risk Assessment:** All identified hazards have been mitigated to acceptable or low residual risk levels. The two medium-risk items (HAZ-02: user behavior, HAZ-19: HealthKit revocation) are inherent to the product category and are managed through disclaimers and graceful degradation respectively.

---

## 4. Benefit-Risk Analysis

The Aura Medical application provides meaningful wellness monitoring benefits (trend tracking, data aggregation, health awareness) that outweigh the identified residual risks, all of which are non-serious and mitigated through multiple design controls. The product does not claim diagnostic capability, does not control treatment, and does not replace medical professionals. The residual risk profile is consistent with IEC 62304 Class B and IMDRF Category II classification.

---

**Approval Signatures:**

| Role                    | Name | Date | Signature |
|-------------------------|------|------|-----------|
| Risk Manager            |      |      |           |
| Quality Manager         |      |      |           |
| Clinical Expert         |      |      |           |
