# Data Contract -- Phenomic Engine v4.2.0

This document defines the shared data types between the three repositories that consume or implement the Phenomic Engine:

| Repository | Role | Engine Version |
|---|---|---|
| `phenomic-engine` | Canonical TypeScript implementation | `4.2.0` |
| `aura-backend` | Cross-validation consumer (vendored) | `4.2.0` (via `vendor/engine/`) |
| `aura-ios` | Swift port for on-device scoring | `4.2.0-swift` |

---

## PhenomicInput (shared)

The input to the engine's `compute()` function. All fields except `age` and `gender` are optional or contain optional sub-fields.

### `wearables`

| Field | Type | Required | Description |
|---|---|---|---|
| `hrv` | `number` | No | Heart rate variability (ms) |
| `rhr` | `number` | No | Resting heart rate (bpm) |
| `sleepScore` | `number` | No | Sleep quality score (0-100) |
| `steps` | `number` | No | Daily step count |

### `labs` (`AllostaticLabs`)

| Field | Type | Required | Description |
|---|---|---|---|
| `homaIr` | `number` | No | HOMA-IR insulin resistance index |
| `hba1c` | `number` | No | Glycated hemoglobin (%) |
| `apoB` | `number` | No | Apolipoprotein B (mg/dL) |
| `triglycerides` | `number` | No | Triglycerides (mg/dL) |
| `hdl` | `number` | No | HDL cholesterol (mg/dL) |
| `hsCrp` | `number` | No | High-sensitivity C-reactive protein (mg/L) |
| `homocysteine` | `number` | No | Homocysteine (umol/L) |
| `ferritin` | `number` | No | Ferritin (ng/mL) |
| `fibrinogen` | `number` | No | Fibrinogen (mg/dL) |
| `cortisol` | `number` | No | Cortisol (ug/dL) |
| `dheaS` | `number` | No | DHEA-S (ug/dL) |

### `physical` (`PhysicalData`)

| Field | Type | Required | Description |
|---|---|---|---|
| `bmi` | `number` | Yes | Body mass index (kg/m^2) |
| `waistHipRatio` | `number` | No | Waist-to-hip ratio |
| `systolicBP` | `number` | No | Systolic blood pressure (mmHg) |
| `diastolicBP` | `number` | No | Diastolic blood pressure (mmHg) |

### `symptoms` (`ClinicalSymptoms`)

All fields use a normalized 0-10 scale, or `null` if not collected.

| Field | Type | Description |
|---|---|---|
| `fatigue` | `number \| null` | General fatigue level |
| `insomnia` | `number \| null` | Sleep difficulty |
| `jointPain` | `number \| null` | Joint pain severity |
| `anxiety` | `number \| null` | Anxiety level |
| `digestiveIssues` | `number \| null` | GI symptom severity |
| `brainFog` | `number \| null` | Cognitive fog severity |
| `lowLibido` | `number \| null` | Libido reduction |
| `postPrandialSlump` | `number \| null` | Post-meal energy crash |

### `genetics` (optional)

| Field | Type | Values | Description |
|---|---|---|---|
| `cyp2c9` | `string` | `Normal`, `Intermediate`, `Poor` | CYP2C9 metabolizer status |
| `akt1` | `string` | `Normal`, `Risk` | AKT1 variant |
| `comt` | `string` | `Val/Val`, `Val/Met`, `Met/Met` | COMT polymorphism |

### Root-level fields

| Field | Type | Required | Description |
|---|---|---|---|
| `age` | `number` | Yes | Patient age in years |
| `gender` | `string` | Yes | `'male'` or `'female'` |
| `ethnicity` | `string` | No | `'African'`, `'Caucasian'`, or `'Other'` |
| `userGoal` | `string` | No | One of: `'FOCUS'`, `'SLEEP'`, `'BALANCE'`, `'PERFORMANCE'`, `'WEIGHT'` |
| `wearableTrends` | `WearableTrends` | No | Rolling average trends for modulation (see below) |

### `wearableTrends` (`WearableTrends`)

Each sub-field (hrv, rhr, sleep, steps) has the same shape:

| Field | Type | Description |
|---|---|---|
| `avg_7d` | `number` | 7-day rolling average |
| `avg_30d` | `number` | 30-day rolling average |
| `avg_all` | `number` (optional) | All-time average |
| `days_total` | `number` (optional) | Total days of data |
| `trend_pct` | `number` | Trend percentage (positive = improving, negative = worsening) |

---

## PhenomicResult (shared)

The output of the engine's `compute()` function.

| Field | Type | Description |
|---|---|---|
| `totalScore` | `number` | Overall allostatic load score (0-100, integer) |
| `slices` | `PhenomicSlice[]` | 5 domain scores (see below) |
| `clinicalWarnings` | `ClinicalWarning[]` (optional) | Non-blocking clinical advisories |
| `dragApplied` | `boolean` | Whether Systemic Drag was applied |
| `precision` | `number` | Data completeness confidence (0-100, integer) |
| `inferenceApplied` | `boolean` | Whether InferenceLayer made estimations |
| `inferredSymptoms` | `string[]` (optional) | List of symptoms inferred from correlations |
| `mostImpactedId` | `string` | Domain ID with lowest score (engine-decided, with tiebreaker) |
| `capsApplied` | `CapsApplied` (optional) | Which biomarker-driven caps were triggered |
| `activityBufferApplied` | `ActivityBufferApplied` (optional) | Steps-based bonus/penalty details |
| `wearableInsights` | `WearableInsights` (optional) | HRV veto + trend modulation visibility |
| `inputValidation` | `{ rejected: string[] }` (optional) | Fields rejected by input validation |
| `auditLog` | `PhenomicAuditLog` (optional) | Mathematical validation trail |

### `PhenomicSlice`

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Domain identifier (e.g., `metabolic`, `cardio`, `sleep`, `nervous`, `inflammatory`) |
| `name` | `string` | Human-readable domain name |
| `value` | `number` | Domain score (0-100) |
| `weight` | `number` | Domain weight (0-1, sums to 1.0) |
| `isGhost` | `boolean` | `true` if domain has no data (Ghost Mode) |
| `color` | `string` | Hex color for UI rendering |
| `inferredFrom` | `string[]` (optional) | Symptoms inferred for this domain |

### `ClinicalWarning`

| Field | Type | Description |
|---|---|---|
| `code` | `string` | Warning code identifier |
| `severity` | `string` | `'critical'` or `'moderate'` |
| `message` | `string` | Human-readable warning message |
| `suggestedDomain` | `string` | Related health domain |
| `evidence` | `string` | Clinical evidence reference |

### `CapsApplied`

| Field | Type | Description |
|---|---|---|
| `cardio` | `string[]` (optional) | Applied caps: `'bloodPressure'`, `'apoB'` |
| `metabolic` | `string[]` (optional) | Applied caps: `'hba1c'`, `'homaIr'`, `'bmi'`, `'whr'` |
| `inflammatory` | `string[]` (optional) | Applied caps: `'hsCrp'` |

### `ActivityBufferApplied`

| Field | Type | Description |
|---|---|---|
| `status` | `string` | `'bonus'`, `'penalty'`, or `'neutral'` |
| `stepsUsed` | `number` (optional) | Step count used for calculation |
| `fallbackUsed` | `boolean` | Whether fallback value was used |
| `domainsAffected.metabolic` | `string` | `'bonus'`, `'penalty'`, or `'none'` |
| `domainsAffected.nervous` | `string` | `'bonus'`, `'penalty'`, or `'none'` |
| `domainsAffected.cardio` | `string` | `'weighted_input'` |

### `WearableInsights`

| Field | Type | Description |
|---|---|---|
| `hrvVeto` | `boolean` (optional) | `true` if HRV < 20ms triggered nervous cap at 40% |
| `trendModulations.nervous_hrv` | `string` (optional) | `'positive'` or `'negative'` |
| `trendModulations.cardio_rhr` | `string` (optional) | `'positive'` or `'negative'` |
| `trendModulations.sleep` | `string` (optional) | `'positive'` or `'negative'` |
| `trendModulations.metabolic_steps` | `string` (optional) | `'positive'` or `'negative'` |

---

## /engine/validate -- Request Schema (Zod)

`POST /engine/validate` -- sent by iOS after on-device scoring.

| Field | Type | Required | Description |
|---|---|---|---|
| `input_hash` | `string` | Yes | SHA-256 hash of the input data |
| `output_score` | `integer` | Yes | iOS-computed total score (0-100) |
| `output_slices` | `array` | Yes | iOS-computed domain slices (see below) |
| `engine_version` | `string` | Yes | iOS engine version (e.g., `"4.2.0-swift"`) |
| `precision` | `integer` | Yes | Precision/confidence score (0-100) |
| `most_impacted_id` | `string` | Yes | Domain ID with lowest score |
| `input_data` | `object` | No | Full `PhenomicInput` for recomputation |

### `output_slices` item

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Domain identifier |
| `value` | `number` | Domain score |
| `weight` | `number` | Domain weight |

---

## /engine/validate -- Response Schema

| Field | Type | Description |
|---|---|---|
| `valid` | `boolean` | `true` if cross-validation passed (delta <= 2) |
| `engine_version` | `string` | The iOS engine version echoed back |
| `backend_engine_version` | `string` | Backend canonical engine version |
| `recorded_at` | `string` | ISO 8601 timestamp |
| `delta` | `number` (optional) | Absolute score difference (present if recomputed) |
| `backend_score` | `number` (optional) | Backend-computed score (present if recomputed) |
| `version_warning` | `string` (optional) | Present if major version mismatch detected |

---

## Version Consistency Rules

1. **Source of truth**: `phenomic-engine/package.json` version field
2. **Backend**: `aura-backend/vendor/engine/package.json` must match (copied via `prebuild` script)
3. **iOS**: `Engine/PhenomicEngine.swift` `engineVersion` must match with `-swift` suffix (e.g., `"4.2.0-swift"`)
4. **Major version mismatch**: If iOS major != backend major, a warning is logged to `ai_audit_log.output_summary` and returned in the response. Validation is NOT blocked.
5. **Cross-validation tolerance**: Score delta <= 2 between iOS and backend is considered valid.

---

## Parity Testing

Test vectors are maintained in `phenomic-engine/parity-vectors.json`. Both the TS engine and Swift port must produce identical results (within floating-point tolerance) for all vectors.

When updating the engine:
1. Update the canonical TS implementation in `phenomic-engine/`
2. Regenerate/update parity vectors
3. Port changes to `aura-ios/Engine/PhenomicEngine.swift`
4. Run parity tests on both platforms
5. Copy updated engine to `aura-backend/vendor/engine/` (automatic via `prebuild`)
6. Bump version in all three locations
