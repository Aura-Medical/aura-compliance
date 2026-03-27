# Verification and Validation Plan

**Document ID:** VV-001
**Revision:** 1.0
**Date:** 2026-03-27
**Product:** Aura Medical iOS Application
**Standard:** IEC 62304:2006+AMD1:2015 Section 5.7 (Software verification)

---

## 1. Scope

This document describes the verification and validation strategy for the Aura Medical iOS application, covering unit tests, integration tests, parity tests, cross-validation procedures, and acceptance criteria. The plan ensures that all requirements defined in TRACEABILITY.md are verified.

## 2. Test Strategy Overview

| Test Level          | Scope                                          | Framework    | Automation |
|---------------------|-------------------------------------------------|-------------|------------|
| Unit Tests          | Engine functions, config, inference, validation  | XCTest      | CI         |
| Parity Tests        | Swift vs. TypeScript engine output equivalence   | XCTest + Vitest | CI     |
| Integration Tests   | Backend cross-validation, Supabase operations    | XCTest      | Manual     |
| Security Tests      | Keychain, cert pinning, biometric auth, PHI blur | XCTest + Manual | Mixed  |
| UI/Manual QA        | Score display, disclaimers, permissions, flows   | Manual      | Manual     |
| Clinical Validation | Score plausibility against medical literature    | Expert review | Manual   |

## 3. Coverage Targets

| Component              | Target Coverage | Rationale                                         |
|------------------------|----------------|----------------------------------------------------|
| Engine/ (core scoring) | **>80%**       | SaMD Class B requires high confidence in scoring logic |
| Security/              | **>70%**       | Security controls must be verified thoroughly       |
| Services/              | **>60%**       | Service orchestration tested via integration        |
| Overall                | **>60%**       | IEC 62304 Class B minimum with risk-based approach  |

## 4. Test Suites

### 4.1 Engine Parity Tests

**Purpose:** Verify that the Swift port of the Phenomic Engine produces identical results to the canonical TypeScript implementation for a defined set of test vectors.

**Source of Truth:** `phenomic-engine/parity-vectors.json` (8 canonical test vectors)

**Test Vectors:**

| Vector | Description                                | Expected Score | Key Assertion           |
|--------|--------------------------------------------|---------------|-------------------------|
| 1      | Complete healthy male, all data             | 78-82         | Full data path          |
| 2      | Complete healthy female, all data           | 75-80         | Gender-specific ranges   |
| 3      | Minimal data (questionnaire only)           | 35-45         | Ghost Mode caps active   |
| 4      | Extreme high-risk biomarkers                | 15-25         | Data caps, systemic drag |
| 5      | Wearable-only (no labs)                     | 50-60         | Inference active         |
| 6      | Lab-only (no wearables)                     | 55-65         | Activity fallback        |
| 7      | Out-of-range inputs (validation edge cases) | Variable      | Validation rejects       |
| 8      | Mixed data with inference needs             | 60-70         | Inference + real data    |

**Acceptance Criteria:**
- Swift `totalScore` must match TypeScript `totalScore` exactly (integer comparison).
- Each domain slice score must match within +/- 1 point.
- All rejected inputs must produce identical `rejected[]` arrays.
- InferredIds arrays must be identical.

**Procedure:**
1. Load `parity-vectors.json` in XCTest.
2. For each vector, construct `EngineComputeRequest` from JSON input.
3. Call `PhenomicEngine.compute(input:config:)` with default config.
4. Assert `totalScore`, each slice score, rejected list, and inferredIds against expected values.

**Related Requirements:** REQ-01, REQ-02, REQ-04, REQ-12, REQ-13, REQ-14, REQ-15, REQ-22, REQ-23

### 4.2 Input Validation Tests

**Purpose:** Verify that `validateInput()` correctly rejects physiologically implausible values and sanitizes input.

**Test Cases:**

| Test Case             | Input                           | Expected Result                          |
|-----------------------|---------------------------------|------------------------------------------|
| VAL-01: Glucose high  | homaIr: 999                    | Rejected: "homaIr: 999 outside [...]"    |
| VAL-02: HRV negative  | hrv: -10                        | Rejected: "hrv: -10 outside [...]"        |
| VAL-03: NaN handling  | rhr: NaN                        | Rejected: "rhr: NaN"                      |
| VAL-04: Age too low   | age: 5                          | Clamped to 18; added to rejected          |
| VAL-05: Age too high  | age: 200                        | Clamped to 120; added to rejected         |
| VAL-06: All valid     | All within range                | valid: true, rejected: []                 |
| VAL-07: BMI nil       | bmi: nil                        | Falls back to 24.5 (neutral BMI)          |
| VAL-08: BP extreme    | systolicBP: 300                 | Rejected: "systolicBP: 300 outside [...]" |
| VAL-09: Steps extreme | steps: 200000                   | Rejected: "steps: 200000 outside [...]"   |
| VAL-10: Multiple bad  | hrv: -1, rhr: 300, cortisol: 99 | All three in rejected array               |

**Acceptance Criteria:**
- Every input field has min/max validation.
- NaN values are caught and rejected.
- Age is clamped (not rejected) to [18, 120].
- BMI falls back to 24.5 when nil.
- `ValidationResult.valid` is `false` when any field rejected.

**Related Requirements:** REQ-01

### 4.3 Inference Layer Tests

**Purpose:** Verify that the Inference Layer correctly estimates missing symptoms using correlation coefficients and discount factors.

**Test Cases:**

| Test Case             | Measured Symptoms       | Expected Behavior                            |
|-----------------------|------------------------|----------------------------------------------|
| INF-01: All present   | All non-nil            | No inference; inferredIds empty               |
| INF-02: Basic infer   | 3 measured, 2 nil      | Missing inferred from correlations             |
| INF-03: Below threshold | Source value < 3     | No propagation (threshold guard)               |
| INF-04: Discount factor | Confidence level 2   | Inferred values reduced by discount factor     |
| INF-05: Real overrides | Target has real data   | Real value preserved, not overwritten          |
| INF-06: Audit trail   | Any inference           | AuditEntry contains source, coefficient, target |
| INF-07: Ghost default | No measured symptoms    | Ghost default (3) used as baseline             |

**Acceptance Criteria:**
- Real data is never overwritten by inference.
- Propagation threshold (>= 3) enforced.
- Discount factors correctly applied per confidence level.
- Audit trail entry created for every inferred value.

**Related Requirements:** REQ-15, REQ-35

### 4.4 Cross-Validation Tests

**Purpose:** Verify that the iOS app correctly sends computation results to the backend for TypeScript engine cross-validation, and that discrepancies are detected.

**Procedure:**

1. **Setup:** Deploy backend with `/api/engine/validate` endpoint active.
2. **Trigger:** Call `PhenomicService.recompute()` with known test data.
3. **Verify Request:** Capture network request to backend; confirm payload includes:
   - `input_hash` (SHA-256)
   - `output_score` (integer 0--100)
   - `output_slices` (5 domain objects)
   - `engine_version` ("4.2.0-swift")
   - `precision` (integer)
   - `most_impacted_id` (string)
   - `input_data` (full input for server-side recomputation)
4. **Verify Response:** Confirm `EngineValidationResponse` includes:
   - `valid: true` (or `false` with `delta` value)
   - `delta` field when cross-validation performed
5. **Discrepancy Test:** Artificially modify Swift engine output by 5 points; verify backend detects `delta > 2` and flags as invalid.

**Acceptance Criteria:**
- Cross-validation fires on every `recompute()` call (fire-and-forget).
- Delta <= 2: validation passes.
- Delta > 2: validation fails, discrepancy logged.
- Network failure does not block user experience.

**Related Requirements:** REQ-04

### 4.5 Security Tests

**Purpose:** Verify all security controls are functional and effective.

| Test Case          | Component                  | Procedure                                               | Expected Result                     |
|--------------------|----------------------------|---------------------------------------------------------|-------------------------------------|
| SEC-01: Keychain   | AuraKeychain               | Save, load, delete cycle; verify accessibility attribute | Data persisted and retrievable       |
| SEC-02: Cert Pin   | CertificatePinning         | Connect to pinned host; connect to unpinned host        | Pinned succeeds; unpinned allowed    |
| SEC-03: Pin fail   | CertificatePinning         | Connect with mismatched pin hash                        | Connection rejected (fail-closed)    |
| SEC-04: Biometric  | BiometricAuthManager       | Enable biometric; background app; return to foreground  | Lock screen appears                  |
| SEC-05: Idle timer | BiometricAuthManager       | Enable biometric; idle 30+ minutes                      | Auto-lock triggers                   |
| SEC-06: PHI blur   | PHIProtectionModifier      | Background app; take screenshot; start screen recording | 30-radius blur applied               |
| SEC-07: Audit log  | EngineAuditLogger          | Compute score; check ai_audit_log table                 | Row created with correct hash/version |
| SEC-08: Audit fail | EngineAuditLogger          | Disconnect network; compute score                       | Score returned; failure logged locally |
| SEC-09: App Attest | AppAttestManager           | First launch on device; verify keychain entry           | Key generated and stored             |
| SEC-10: Logout     | AuthViewModel.signOut()    | Sign out; verify SwiftData and Keychain cleared         | All local PHI deleted                |

**Related Requirements:** REQ-03, REQ-05, REQ-06, REQ-07, REQ-08, REQ-09, REQ-11, REQ-18, REQ-19

### 4.6 Clinical Validation

**Purpose:** Verify that engine scores are clinically plausible and consistent with medical literature on allostatic load.

**Procedure:**

1. **Expert Review:** Clinical advisor reviews scoring algorithm, reference ranges, and biomarker weights against published allostatic load literature (McEwen 1998, Juster et al. 2010, Seeman et al. 2001).
2. **Case Studies:** Process 20+ representative patient profiles (anonymized) through the engine and review scores for clinical plausibility.
3. **Boundary Analysis:** Verify that:
   - Perfectly healthy biomarkers produce score >= 75.
   - All biomarkers at critical thresholds produce score <= 30.
   - Mixed profiles produce intermediate scores proportional to risk burden.
4. **Gender Sensitivity:** Verify gender-specific reference ranges (WHR, HDL, ferritin, DHEA-S) produce clinically appropriate score differences.

**Acceptance Criteria:**
- Clinical advisor signs off on algorithm and reference ranges.
- 90% of case studies produce clinically plausible scores (expert judgment).
- No false "excellent" (>80) for clearly unhealthy profiles.
- No false "critical" (<20) for clearly healthy profiles.

**Related Requirements:** REQ-02, REQ-12, REQ-24

## 5. Test Execution Procedures

### 5.1 Running Parity Tests (Swift)

```bash
# From aura-ios/ repository root
cd /Users/alexandretalmeida/aura-wearables/aura-ios
xcodebuild test \
  -scheme AuraMedical \
  -destination 'platform=iOS,name=<device_name>' \
  -testPlan EngineParity \
  -resultBundlePath TestResults/parity.xcresult
```

### 5.2 Running Parity Tests (TypeScript)

```bash
# From phenomic-engine/ repository root
cd /Users/alexandretalmeida/aura-wearables/phenomic-engine
npm test -- --run parity
```

### 5.3 Running All Swift Tests

```bash
cd /Users/alexandretalmeida/aura-wearables/aura-ios
xcodebuild test \
  -scheme AuraMedical \
  -destination 'platform=iOS,name=<device_name>' \
  -resultBundlePath TestResults/full.xcresult
```

### 5.4 Running Backend Cross-Validation Test

```bash
# From aura-backend/ repository root
cd /Users/alexandretalmeida/aura-wearables/aura-backend
npm test -- --run engine-validate
```

### 5.5 Measuring Code Coverage

```bash
cd /Users/alexandretalmeida/aura-wearables/aura-ios
xcodebuild test \
  -scheme AuraMedical \
  -destination 'platform=iOS,name=<device_name>' \
  -enableCodeCoverage YES \
  -resultBundlePath TestResults/coverage.xcresult

# Extract coverage report
xcrun xccov view --report TestResults/coverage.xcresult
```

## 6. Test Environment

| Component         | Specification                                   |
|-------------------|-------------------------------------------------|
| Xcode             | 16.x (latest stable)                            |
| iOS Deployment    | 17.0 minimum                                    |
| Test Device       | Physical iOS device (NOT simulator -- per project policy) |
| Node.js           | 20 LTS                                          |
| TypeScript Engine  | Vitest                                          |
| Backend           | Hono v4 on Node 20                              |
| Database          | Supabase (PostgreSQL 15+)                        |

## 7. Defect Management

| Severity | Definition                                          | Response Time | Resolution Target |
|----------|-----------------------------------------------------|---------------|-------------------|
| Critical | Score calculation produces clinically dangerous result | 4 hours       | 24 hours          |
| High     | Security control failure; PHI exposure                | 8 hours       | 48 hours          |
| Medium   | Score inaccuracy > 5 points from expected            | 24 hours      | 1 week            |
| Low      | UI display issue; non-blocking error                  | 1 week        | Next release      |

## 8. Release Criteria

The following criteria must be met before any release:

1. All parity test vectors pass (0 failures).
2. Input validation test suite passes (0 failures).
3. Engine code coverage >= 80%.
4. Overall code coverage >= 60%.
5. No Critical or High severity open defects.
6. Cross-validation delta <= 2 for all test vectors.
7. Security test suite passes (all SEC-01 through SEC-10).
8. Clinical advisor sign-off on scoring algorithm.
9. All disclaimers present and readable on score screens.
10. Privacy usage descriptions present for all permissions.

---

**Approval Signatures:**

| Role                    | Name | Date | Signature |
|-------------------------|------|------|-----------|
| Quality Manager         |      |      |           |
| Test Lead               |      |      |           |
| Software Development Lead |    |      |           |
