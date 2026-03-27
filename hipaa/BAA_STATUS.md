# Business Associate Agreement (BAA) Status Tracker

**Document ID:** AURA-POL-BAA-001
**Version:** 1.0
**Effective Date:** 2026-03-27
**Last Reviewed:** 2026-03-27
**Next Review:** 2026-09-27
**Owner:** Legal / Compliance, Auramedical Tecnologia
**Classification:** Internal

---

## 1. Purpose

This document tracks the status of Business Associate Agreements (BAAs) and equivalent data processing agreements with all third-party vendors that process, store, or transmit Protected Health Information (PHI) or Personally Identifiable Information (PII) on behalf of Aura Medical. This tracker ensures compliance with:

- **HIPAA** (45 CFR 164.502(e), 164.504(e)) — BAA requirements for business associates
- **LGPD** (Lei 13.709/2018, Art. 39) — Data processor obligations and agreements

## 2. What Constitutes a Business Associate

### 2.1 HIPAA Definition

Under HIPAA, a **Business Associate** is any person or entity that:

- Creates, receives, maintains, or transmits PHI on behalf of a covered entity
- Provides services to a covered entity involving the use or disclosure of PHI

A BAA is a written contract that:

- Establishes the permitted uses and disclosures of PHI
- Requires the business associate to implement appropriate safeguards
- Requires reporting of security incidents and breaches
- Ensures PHI is returned or destroyed at termination
- Authorizes termination if the business associate violates the agreement

### 2.2 LGPD Equivalent (Data Processor Agreement)

Under LGPD, a **Data Processor (Operador)** is any natural or legal person that processes personal data on behalf of the **Data Controller (Controlador)**. While LGPD does not mandate a specific "BAA" document, Art. 39 requires:

- Written instructions from the controller to the processor
- Documentation of processing activities
- Technical and organizational security measures
- Sub-processor restrictions
- Data breach notification obligations
- Data deletion or return upon contract termination

A **Data Processing Agreement (DPA)** serves the equivalent function of a BAA under LGPD.

---

## 3. Vendor Agreement Status

### 3.1 Master Status Table

| Vendor | Service | PHI Processed? | Data Categories | BAA Status | DPA (LGPD) Status | Action Required | Priority | Deadline |
|--------|---------|----------------|-----------------|------------|-------------------|-----------------|----------|----------|
| **Supabase** | Database, Auth, Storage, Edge Functions | **Yes** — All PHI (scores, labs, biometrics, profiles, questionnaires) | Health scores, lab results, biometric data, PII, audit logs | Available on Pro plan; **not yet signed** | Supabase DPA available | **Sign BAA and DPA** via Supabase Dashboard | **CRITICAL** | Immediate |
| **Render** | Backend hosting (Hono API) | **Transit only** — PHI passes through via TLS; no persistent storage | API request/response payloads (encrypted in transit) | Check eligibility — contact sales | Check DPA availability | **Contact Render sales** to assess BAA/DPA eligibility | **HIGH** | 2 weeks |
| **Google Cloud** | OAuth (Google Sign-In) | **No** — Email address only for authentication | Email address (PII, not PHI) | N/A — No PHI processed | Standard Google ToS covers | Review Google Cloud DPA for PII coverage | LOW | Quarterly review |
| **Apple** | HealthKit, Sign-In with Apple | **Yes** — Health data accessed via HealthKit APIs | Heart rate, steps, sleep, HRV, respiratory rate, blood oxygen | Apple Developer Program License Agreement includes health data provisions | Apple's standard privacy terms apply | **Verify** that DPLA Section 3.3.28 (HealthKit) covers BAA requirements | **MEDIUM** | 1 month |
| **Apple** | App Store (distribution) | **No** — App binary only | App metadata, crash logs (no PHI) | Standard Apple Developer Agreement | Standard terms | None — no PHI in scope | — | — |
| **OpenAI / Anthropic** | Future AI features (chat, insights) | **TBD** — Will depend on implementation | Potentially: user queries, health context | **TBD** — Must assess before integration | **TBD** | **Assess BAA/DPA requirements before any integration** | LOW | Before integration |

### 3.2 Status Definitions

| Status | Meaning |
|--------|---------|
| **Signed** | BAA/DPA executed and on file |
| **Available** | Vendor offers BAA/DPA; not yet signed by Aura Medical |
| **In Progress** | BAA/DPA negotiation or review underway |
| **Check Eligibility** | Unknown whether vendor offers BAA/DPA; needs investigation |
| **N/A** | No PHI processed; BAA not required |
| **TBD** | Future integration; requirements not yet determined |

---

## 4. Vendor Details

### 4.1 Supabase

**Criticality:** CRITICAL — Primary data store for all PHI

| Attribute | Detail |
|-----------|--------|
| Service | PostgreSQL database, Auth (GoTrue), Storage, Edge Functions |
| Plan | Pro ($25/month) |
| Data residency | AWS us-east-1 (Virginia) |
| PHI categories | All — health scores, lab results, biometric summaries, user profiles, audit logs |
| Encryption at rest | AES-256 (AWS default) |
| Encryption in transit | TLS 1.2+ (certificate pinning enforced by iOS client) |
| BAA availability | Available on Pro plan via Dashboard (Settings > Legal) |
| HIPAA readiness | Supabase offers HIPAA-compliant configuration on Pro+ plans |

**Required Actions:**

1. Sign BAA via Supabase Dashboard (Settings > Legal > Sign BAA)
2. Enable HIPAA-compliant project configuration
3. Verify pgaudit is active for all PHI tables
4. Confirm backup encryption settings
5. Document signed BAA with effective date

### 4.2 Render

**Criticality:** HIGH — Backend API processes PHI in transit

| Attribute | Detail |
|-----------|--------|
| Service | Hono API hosting (Node.js) |
| Plan | Starter ($7/month) |
| Data handling | PHI in transit only — no persistent PHI storage |
| Encryption in transit | TLS 1.2+ (certificate pinning enforced by iOS client) |
| Logging | Application logs may contain request metadata (no PHI — enforced by audit middleware with PHI redaction) |
| BAA availability | Unknown — requires inquiry |

**Required Actions:**

1. Contact Render support/sales to inquire about BAA availability
2. If BAA not available, evaluate:
   - Can PHI transit be classified as "conduit exception" under HIPAA?
   - Are additional encryption measures needed (e.g., field-level encryption)?
   - Should we migrate to a HIPAA-eligible hosting provider (AWS, GCP, Azure)?
3. Review Render's DPA for LGPD compliance
4. Verify that application logs do not contain PHI (confirm Pino PHI redaction middleware)

**Conduit Exception Note:** Under HIPAA, a "conduit" (like a telephone company or ISP) that merely transmits PHI is not a business associate. If Render only transmits encrypted PHI without access to the decryption keys and without persistent storage, the conduit exception may apply. However, this determination should be reviewed by legal counsel.

### 4.3 Apple (HealthKit)

**Criticality:** MEDIUM — Source of biometric health data

| Attribute | Detail |
|-----------|--------|
| Service | HealthKit API, Sign-In with Apple |
| Data flow | iOS app reads HealthKit data with user permission; data processed locally and synced to Supabase |
| PHI categories | Heart rate, steps, sleep analysis, HRV, respiratory rate, blood oxygen |
| HealthKit restrictions | DPLA Section 3.3.28 — strict requirements on health data handling |
| Apple's position | Apple does not consider itself a business associate; HealthKit data handling is the developer's responsibility |

**Required Actions:**

1. Review Apple Developer Program License Agreement Section 3.3.28 (HealthKit)
2. Verify compliance with all HealthKit data handling requirements:
   - No advertising use of HealthKit data
   - No sale of HealthKit data to third parties
   - Data not stored in iCloud (Aura uses Supabase, not iCloud)
   - Clear privacy policy disclosure
3. Document that Apple's agreement addresses health data obligations
4. Ensure App Privacy nutrition labels accurately reflect HealthKit data usage

### 4.4 Future AI Providers (OpenAI / Anthropic)

**Criticality:** LOW (currently) — No integration exists yet

**Assessment Required Before Integration:**

1. Determine what data will be sent to the AI provider:
   - If any PHI (health scores, lab results, symptoms): BAA is REQUIRED
   - If only de-identified/anonymized data: BAA may not be required, but DPA is recommended
2. Both OpenAI and Anthropic offer BAAs on enterprise plans
3. Evaluate on-device AI alternatives (Apple Intelligence, Core ML) to avoid PHI transmission
4. If cloud AI is needed, consider:
   - PHI de-identification before transmission
   - Field-level encryption
   - Data processing agreements under LGPD

---

## 5. Review Schedule

### 5.1 Regular Reviews

| Review Type | Frequency | Scope | Responsible |
|-------------|-----------|-------|-------------|
| **BAA status check** | Quarterly | Verify all BAAs are current and signed | Legal / Compliance |
| **Vendor security review** | Annually | Review each vendor's security posture, SOC 2 reports, penetration test results | Security Officer |
| **New vendor assessment** | Before onboarding | Full BAA/DPA assessment for any new vendor processing PHI or PII | Legal + Engineering |
| **Agreement renewal** | Per contract terms | Ensure BAAs are renewed before expiration | Legal |

### 5.2 Triggering Events for Immediate Review

- New vendor onboarded that may process PHI
- Vendor changes terms of service or privacy policy
- Vendor experiences a security breach
- Aura Medical changes the type of data shared with a vendor
- Regulatory changes affecting BAA/DPA requirements

---

## 6. Risk Assessment Matrix

| Vendor | Risk if No BAA | Likelihood of PHI Exposure | Overall Risk |
|--------|---------------|---------------------------|--------------|
| Supabase | **Critical** — All PHI stored here | High — direct database access | **Critical** |
| Render | **High** — PHI transits through | Medium — transit only, TLS encrypted | **High** |
| Apple (HealthKit) | **Medium** — Data read from HealthKit | Low — data flows from Apple to app, not reverse | **Medium** |
| Google (OAuth) | **Low** — No PHI involved | Very Low — email only | **Low** |
| Future AI | **TBD** — Depends on implementation | TBD | **TBD** |

---

## 7. BAA Execution Checklist

When signing a new BAA, complete the following:

- [ ] Verify vendor is listed in this tracker
- [ ] Review BAA terms with legal counsel
- [ ] Confirm BAA covers all PHI categories shared with vendor
- [ ] Verify breach notification timeline aligns with our [BREACH_RESPONSE.md](BREACH_RESPONSE.md)
- [ ] Confirm data return/destruction provisions align with [DATA_RETENTION.md](DATA_RETENTION.md)
- [ ] Sign BAA (electronic or physical)
- [ ] Store signed BAA in secure document repository
- [ ] Update this tracker with signed date and effective date
- [ ] Set calendar reminder for annual review
- [ ] Notify Security Officer that BAA is in place

---

## 8. Signed Agreements Register

| Vendor | Agreement Type | Signed Date | Effective Date | Expiration | Signed By | Document Location |
|--------|---------------|-------------|----------------|------------|-----------|-------------------|
| *No agreements signed yet* | | | | | | |

> **Action Item:** Sign Supabase BAA as first priority. Update this table upon execution.

---

## 9. Related Documents

- [BREACH_RESPONSE.md](BREACH_RESPONSE.md) — Breach response procedures (vendor breach notifications)
- [DATA_RETENTION.md](DATA_RETENTION.md) — Data retention and destruction policies
- [ACCOUNT_DELETION.md](ACCOUNT_DELETION.md) — Account deletion procedures affecting vendor data
- `AuraMedical/Security/CertificatePinning.swift` — TLS certificate pinning for vendor connections
- `AuraMedical/Services/AuraBackendClient.swift` — Backend client with cert pinning and JWT auth

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-27 | Legal / Compliance | Initial version |
