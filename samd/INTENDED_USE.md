# Intended Use Statement

**Document ID:** IU-001
**Revision:** 1.0
**Date:** 2026-03-27
**Product:** Aura Medical iOS Application
**Manufacturer:** Auramedical Tecnologia
**Classification:** SaMD Class B (IEC 62304), IMDRF Risk Category II

---

## 1. Product Description

Aura Medical is a mobile health application for iOS that calculates an **allostatic load score** (0--100) across five physiological health domains:

| Domain         | Key Biomarkers                                                  |
|----------------|-----------------------------------------------------------------|
| Metabolic      | HOMA-IR, HbA1c, BMI, waist-hip ratio, TG/HDL ratio             |
| Cardiovascular | Systolic/diastolic BP, ApoB, resting heart rate                 |
| Sleep          | Total duration, deep/REM/light percentages, efficiency          |
| Nervous        | HRV, cortisol, DHEA-S, stress questionnaire symptoms            |
| Inflammatory   | hs-CRP, homocysteine, ferritin, fibrinogen                      |

The Phenomic Engine (version 4.2.0-swift) processes inputs from multiple data sources, applies evidence-based normalization and weighting, and produces a composite wellness score with per-domain breakdowns.

## 2. Intended Use

Aura Medical is intended to provide **general wellness and self-monitoring information** to healthy adults. The software aggregates biometric, wearable, laboratory, and self-reported data to compute a numerical wellness score that helps users track trends in their overall physiological state over time.

### 2.1 What the Software Does

- Collects health data from Apple HealthKit (HRV, resting heart rate, steps, sleep architecture), user-entered lab results, biometric measurements, and lifestyle questionnaires.
- Validates all numeric inputs against physiological ranges (e.g., glucose capped at clinically plausible bounds) to reject implausible or corrupted data.
- Computes a deterministic allostatic load score (0--100) using a published, versioned algorithm (Phenomic Engine).
- Presents per-domain scores (metabolic, cardiovascular, sleep, nervous, inflammatory) with a five-tier qualitative scale (critical, warning, fair, good, excellent).
- Infers missing data points using an evidence-based correlation matrix with explicit discount factors, clearly marking inferred values.
- Applies Ghost Mode fallbacks when entire data categories are absent to produce conservative (safe-pessimism) estimates.
- Logs every computation to an immutable audit trail (ai_audit_log) with SHA-256 input hashes for reproducibility.
- Cross-validates local (Swift) engine computations against the canonical TypeScript engine on the backend, flagging discrepancies exceeding a delta of 2 points.

### 2.2 What the Software Does NOT Do

- **Does not diagnose** any medical condition, disease, or disorder.
- **Does not prescribe** treatments, medications, or therapeutic interventions.
- **Does not replace** the judgment, examination, or advice of qualified medical professionals.
- **Does not provide** emergency or urgent medical alerts.
- **Does not control** any therapeutic device, drug delivery system, or clinical workflow.
- **Does not make** treatment decisions or recommend specific clinical actions.
- **Does not claim** to detect, prevent, or treat any specific disease.

## 3. Target Users

| User Category         | Description                                                        |
|-----------------------|--------------------------------------------------------------------|
| Primary users         | Healthy adults (18--120 years) seeking self-monitoring wellness information |
| Use environment       | Personal, non-clinical, consumer setting                            |
| Technical proficiency | General smartphone literacy; no medical training required            |
| Geographic scope      | Initially Brazil (PT-BR interface), global availability planned      |

The software is **not intended** for use by:
- Patients under active medical treatment relying on the score for treatment decisions
- Healthcare professionals as a clinical diagnostic tool
- Pediatric populations (under 18 years)
- Emergency or acute care settings

## 4. Classification Rationale

### 4.1 IEC 62304 Classification: Class B

Per IEC 62304:2006+AMD1:2015, software is classified based on severity of harm from software failure:

| Class | Harm Level       | Applicability                                    |
|-------|-----------------|--------------------------------------------------|
| A     | No injury        | Not applicable (score could influence behavior)   |
| **B** | **Non-serious**  | **Applicable: incorrect score may cause concern or complacency, but no direct physical harm** |
| C     | Serious/death    | Not applicable (no direct treatment control)      |

**Justification:** A software failure (e.g., incorrect score calculation) could lead a user to unnecessary concern or false reassurance about their wellness. However, the software explicitly disclaims diagnostic capability, does not control any treatment, and all screens include disclaimers directing users to consult healthcare professionals. The risk of serious injury is mitigated to non-serious through design controls.

### 4.2 IMDRF Risk Categorization: Category II

Per the IMDRF SaMD Framework (N12), using the significance-of-information matrix:

| Factor                        | Assessment                           |
|-------------------------------|--------------------------------------|
| State of healthcare situation | Non-serious (wellness monitoring)     |
| Significance of information   | Informs (does not drive or treat)     |
| **Risk Category**             | **II**                                |

## 5. Regulatory Pathway

### 5.1 ANVISA (Brazil)

Under ANVISA RDC 185/2001 and the forthcoming SaMD guidance, Aura Medical is positioned as a **Class I/II wellness product**. The application provides informational wellness scores and does not claim to diagnose or treat any condition registered in ICD-10/ICD-11. The intended use aligns with ANVISA's general wellness category, which may exempt the product from full medical device registration provided all marketing materials and in-app disclosures maintain the wellness framing.

### 5.2 FDA (United States)

Under FDA guidance "General Wellness: Policy for Low Risk Devices" (September 2019), Aura Medical qualifies as a **general wellness product** because it:

1. Is intended for general wellness use (maintaining or encouraging a healthy lifestyle)
2. Presents low risk to user safety (informational only, no treatment decisions)
3. Does not make disease-specific claims

If the product later makes disease-specific claims, it would require 510(k) clearance or De Novo classification as a Class II SaMD.

### 5.3 EU MDR

Under EU MDR 2017/745 Article 2(1) and MDCG 2019-11, the software would be classified as a wellness application outside the scope of MDR, provided no diagnostic claims are made. If diagnostic claims are added, classification under Rule 11 (Class IIa minimum) would apply.

## 6. Data Sources

| Source               | Data Type                                    | Collection Method           |
|----------------------|----------------------------------------------|-----------------------------|
| Apple HealthKit      | HRV, resting heart rate, steps, sleep stages | Automatic via HealthKit API |
| Lab Results          | Blood biomarkers (HOMA-IR, HbA1c, hs-CRP, ApoB, etc.) | Manual entry or Lab OCR     |
| Questionnaire        | Lifestyle, symptoms, stress, sleep quality   | In-app questionnaire        |
| Physical Measurements | Height, weight, waist/hip circumference, blood pressure | Manual entry               |

All data processing occurs locally on the user's device. The Phenomic Engine is a pure, deterministic function: same input + same configuration = same output, always.

## 7. Disclaimers

The following disclaimers are displayed within the application:

> **This application is for informational and wellness purposes only. It does not provide medical advice, diagnosis, or treatment. Always consult a qualified healthcare professional before making any health decisions. The allostatic load score is an estimate based on available data and should not be used as the sole basis for any medical decision.**

---

**Approval Signatures:**

| Role                    | Name | Date | Signature |
|-------------------------|------|------|-----------|
| Quality Manager         |      |      |           |
| Regulatory Affairs      |      |      |           |
| Software Development Lead |    |      |           |
