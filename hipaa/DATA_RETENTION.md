# Data Retention Policy

**Document ID:** AURA-POL-DR-001
**Version:** 1.0
**Effective Date:** 2026-03-27
**Last Reviewed:** 2026-03-27
**Next Review:** 2026-09-27
**Owner:** Data Protection Officer, Auramedical Tecnologia
**Classification:** Internal

---

## 1. Purpose

This document defines the data retention periods, legal justifications, and deletion procedures for all categories of data processed by the Aura Medical platform. This policy ensures compliance with:

- **HIPAA** (45 CFR 164.530(j)) — 6-year retention for policies and documentation
- **LGPD** (Lei 13.709/2018, Art. 15-16) — Data must be deleted when purpose is fulfilled
- **Apple App Store Guidelines** — Data minimization and user deletion rights
- **CFR 21 Part 11** — Electronic records for SaMD (Software as a Medical Device)

## 2. Scope

This policy covers all data collected, processed, and stored by:

- Aura Medical iOS application (local SwiftData + Keychain)
- Supabase database (PostgreSQL)
- Backend API (Hono on Render — transit/processing only)
- Third-party services (HealthKit, authentication providers)

---

## 3. Retention Schedule

### 3.1 Master Retention Table

| Data Type | Database Table/Store | Retention Period | Legal Basis | Deletion Method | Automated? |
|-----------|---------------------|-----------------|-------------|-----------------|------------|
| Daily phenomic scores | `daily_phenomic_scores` | 2 years from creation | Legitimate interest — clinical history for trend analysis (LGPD Art. 7, V) | Automated purge via scheduled SQL | Yes |
| Audit logs (SaMD) | `ai_audit_log` | 7 years from creation | HIPAA 164.530(j) — documentation retention; CFR 21 Part 11 — electronic records | Anonymized (user_id nulled), never deleted | Yes (anonymization at account deletion) |
| User profile | `user_profiles` | Life of account + 30 days | Contractual necessity — required for service delivery (LGPD Art. 7, V) | Soft-delete on account deletion; hard-delete after 30-day hold | Yes (cron job) |
| Lab results | `biomarker_results` | 2 years from result date | Legitimate interest — clinical history (LGPD Art. 7, V); professional record-keeping | Automated purge via scheduled SQL | Yes |
| HealthKit summaries | `healthkit_summaries` | 1 year from collection date | Consent (LGPD Art. 7, I) — user-initiated sync | Automated purge via scheduled SQL | Yes |
| Questionnaire responses | `questionnaire_responses` | 2 years from submission | Legitimate interest — engine input history for score continuity (LGPD Art. 7, V) | Automated purge via scheduled SQL | Yes |
| Deleted account data | `user_profiles` (soft-deleted) | 30 days from deletion request | LGPD Art. 16 — right to deletion; grace period for accidental deletion | Hard-delete via cron job | Yes |
| Push notification tokens | `push_tokens` | Life of account | Contractual necessity — required for notification delivery (LGPD Art. 7, V) | Deleted on logout or account deletion | Yes |
| Chat messages | `chat_messages` | 2 years from creation | Legitimate interest — clinical communication record (LGPD Art. 7, V) | Soft-delete; hard-delete after retention period | Yes |
| Order/prescription data | `orders` | 5 years from order date | Legal obligation — Brazilian healthcare record-keeping (Resolucao CFM 1.821/2007) | Archived to cold storage after 5 years | Semi-auto |
| Local SwiftData (iOS) | Device storage | Life of app installation | Local cache — mirrors server data | Deleted on account deletion or app uninstall | Yes |
| Keychain data (iOS) | iOS Keychain | Life of account | Session management — tokens and preferences | Cleared on sign-out or account deletion | Yes |
| pgaudit logs | `pgaudit` extension logs | 7 years | HIPAA compliance — database operation audit trail | Log rotation to archival storage | Semi-auto |

### 3.2 Retention Period Justifications

#### 2-Year Retention (Clinical Data)

Clinical data (scores, lab results, questionnaires, chat messages) is retained for 2 years based on:

- **Clinical utility:** 2 years provides sufficient history for health trend analysis and score continuity
- **Data minimization:** Longer retention is not justified for a wellness/health-scoring application
- **LGPD proportionality:** Retention must be proportional to the purpose (Art. 6, III)
- **User expectation:** Users reasonably expect recent health history to be available

#### 7-Year Retention (Audit Logs)

Audit logs are retained for 7 years based on:

- **HIPAA requirement:** 45 CFR 164.530(j) requires retention of policies and documentation for 6 years from creation or last effective date
- **SaMD compliance:** CFR 21 Part 11 requires electronic records to be maintained for the life of the product plus additional period
- **Anonymization:** After account deletion, audit logs are anonymized (user_id set to null UUID) but retained for compliance
- **Legal defense:** Audit trail may be needed for legal proceedings within statute of limitations

#### 5-Year Retention (Orders/Prescriptions)

- **Brazilian law:** Resolucao CFM 1.821/2007 requires medical records to be maintained for a minimum of 20 years; however, order metadata (not full medical records) has a shorter retention requirement
- **Legal obligation:** LGPD Art. 7, II — processing necessary for legal or regulatory compliance

#### 1-Year Retention (HealthKit Summaries)

- **Data minimization:** Raw HealthKit data is processed into scores; retaining summaries beyond 1 year provides diminishing analytical value
- **Storage efficiency:** HealthKit data is high-volume; shorter retention reduces storage costs
- **User control:** Users can re-sync from HealthKit at any time

---

## 4. Automated Purge Implementation Plan

### 4.1 Architecture

Data purging is implemented as scheduled PostgreSQL functions executed via Supabase's `pg_cron` extension.

### 4.2 Purge Functions

#### 4.2.1 Daily Purge Job (runs at 03:00 UTC daily)

```sql
-- Create the purge function
CREATE OR REPLACE FUNCTION purge_expired_data()
RETURNS void AS $$
BEGIN
  -- Purge phenomic scores older than 2 years
  DELETE FROM daily_phenomic_scores
  WHERE created_at < NOW() - INTERVAL '2 years';

  -- Purge lab results older than 2 years
  DELETE FROM biomarker_results
  WHERE result_date < NOW() - INTERVAL '2 years';

  -- Purge HealthKit summaries older than 1 year
  DELETE FROM healthkit_summaries
  WHERE collected_at < NOW() - INTERVAL '1 year';

  -- Purge questionnaire responses older than 2 years
  DELETE FROM questionnaire_responses
  WHERE submitted_at < NOW() - INTERVAL '2 years';

  -- Purge chat messages older than 2 years (already soft-deleted)
  DELETE FROM chat_messages
  WHERE deleted_at IS NOT NULL
    AND deleted_at < NOW() - INTERVAL '2 years';

  -- Hard-delete accounts past 30-day hold
  -- (See ACCOUNT_DELETION.md for full flow)
  DELETE FROM user_profiles
  WHERE deleted_at IS NOT NULL
    AND deleted_at < NOW() - INTERVAL '30 days';

  -- Log the purge operation
  INSERT INTO ai_audit_log (operation, engine_version, created_at)
  VALUES ('data_purge', 'system', NOW());
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Schedule via pg_cron
SELECT cron.schedule(
  'daily-data-purge',
  '0 3 * * *',
  'SELECT purge_expired_data()'
);
```

#### 4.2.2 Audit Log Anonymization (on account deletion)

```sql
-- Anonymize audit logs when a user deletes their account
-- Called by the delete_my_account() RPC
UPDATE ai_audit_log
SET user_id = '00000000-0000-0000-0000-000000000000'
WHERE user_id = [deleted_user_id];
```

#### 4.2.3 Order Archival (quarterly)

```sql
-- Archive orders older than 5 years to cold storage
-- This is a semi-automated process requiring manual verification
CREATE OR REPLACE FUNCTION archive_old_orders()
RETURNS void AS $$
BEGIN
  -- Move to archive table
  INSERT INTO orders_archive
  SELECT * FROM orders
  WHERE order_date < NOW() - INTERVAL '5 years';

  -- Delete from active table
  DELETE FROM orders
  WHERE order_date < NOW() - INTERVAL '5 years';
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### 4.3 Monitoring

- Purge jobs log results to `ai_audit_log` with operation type `data_purge`
- Failed purge jobs trigger alerts via Supabase webhook
- Monthly verification: compare row counts before/after purge windows

### 4.4 iOS Local Data Purge

Local SwiftData stores mirror server data and follow the same retention periods:

- `SyncManager` removes local records that are no longer present on the server during sync
- On account deletion, `AuthViewModel.signOut()` clears all SwiftData containers and Keychain entries
- See [ACCOUNT_DELETION.md](ACCOUNT_DELETION.md) for the complete local data cleanup flow

---

## 5. LGPD Data Subject Rights Procedures

### 5.1 Right to Access (Art. 18, II)

**Process:**

1. User requests via Settings > "Meus Dados" or email to privacidade@auramedical.com
2. Verify identity via authenticated session or email verification
3. Generate data export within 15 days (LGPD Art. 19, II)
4. Provide data in structured, machine-readable format (JSON)
5. Data export includes all categories in Section 3.1

### 5.2 Right to Correction (Art. 18, III)

**Process:**

1. User updates profile data directly in-app (Settings > Profile)
2. For data that cannot be self-corrected (e.g., lab results), user contacts support
3. Corrections are applied within 15 days
4. Correction is logged in `ai_audit_log`

### 5.3 Right to Deletion (Art. 18, VI)

**Process:**

1. User initiates account deletion in-app (Settings > "Apagar minha conta")
2. Full deletion flow documented in [ACCOUNT_DELETION.md](ACCOUNT_DELETION.md)
3. 30-day grace period for account recovery
4. After 30 days, all personal data is permanently deleted
5. Audit logs are anonymized, not deleted (legal obligation exception — LGPD Art. 16, I)

### 5.4 Right to Data Portability (Art. 18, V)

**Process:**

1. User requests via Settings > "Exportar Dados" or email
2. Generate portable data package within 15 days
3. Format: JSON file containing all user data categories
4. Delivered via secure download link (expires in 24 hours)

### 5.5 Right to Revoke Consent (Art. 18, IX)

**Process:**

1. HealthKit sync consent: User can disable via Settings > "HealthKit" toggle
2. Push notification consent: User can disable via iOS Settings
3. Account consent: User can delete account (see Section 5.3)
4. Revoking consent does not affect data processed before revocation (LGPD Art. 8, Par. 5)

### 5.6 Request Tracking

All data subject requests are tracked in an internal register:

| Field | Description |
|-------|-------------|
| Request ID | Unique identifier |
| Date received | Timestamp |
| Data subject | User ID (anonymized in log after completion) |
| Request type | Access / Correction / Deletion / Portability / Revocation |
| Status | Received / In Progress / Completed / Denied (with justification) |
| Response date | Timestamp of response |
| Response method | In-app / Email / Download link |

---

## 6. Exceptions to Deletion

Per LGPD Art. 16, data may be retained beyond the stated period for:

1. **Legal or regulatory obligation** (Art. 16, I) — Audit logs retained for 7 years per HIPAA
2. **Research purposes** (Art. 16, II) — Only with anonymization; not currently applicable
3. **Transfer to third party** (Art. 16, III) — Only with legal basis; not currently applicable
4. **Legitimate interest of the controller** (Art. 16, IV) — Aggregated, anonymized analytics

---

## 7. Data Destruction Methods

| Method | Description | Used For |
|--------|-------------|----------|
| **SQL DELETE** | Row-level deletion from PostgreSQL | Most data types |
| **Anonymization** | Replace identifying fields with null/zero UUIDs | Audit logs |
| **Soft-delete** | Set `deleted_at` timestamp, exclude from queries | User profiles, chat messages |
| **Hard-delete** | Permanent SQL DELETE after soft-delete hold period | Expired soft-deleted records |
| **SwiftData clear** | Delete ModelContainer and recreate | Local iOS data |
| **Keychain clear** | Delete all items with app's access group | Session tokens, preferences |
| **Archive** | Move to separate table/database with restricted access | Orders after 5 years |

---

## 8. Compliance Verification

### 8.1 Internal Audits

- **Quarterly:** Verify purge jobs are running and removing data on schedule
- **Quarterly:** Sample check — verify no records exist past their retention period
- **Annually:** Full review of retention periods against current legal requirements

### 8.2 Documentation

- Purge job execution logs retained in `ai_audit_log`
- Data subject request register maintained per Section 5.6
- Annual retention policy review documented and signed

---

## 9. Related Documents

- [BREACH_RESPONSE.md](BREACH_RESPONSE.md) — Breach response procedures
- [BAA_STATUS.md](BAA_STATUS.md) — Vendor agreements affecting data handling
- [ACCOUNT_DELETION.md](ACCOUNT_DELETION.md) — Complete account deletion flow
- `AuraMedical/Security/EngineAuditLogger.swift` — Audit log implementation
- `AuraMedical/LocalData/SyncManager.swift` — Local data sync and cleanup

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-27 | Data Protection Officer | Initial version |
