# Account Deletion Flow

**Document ID:** AURA-POL-AD-001
**Version:** 1.0
**Effective Date:** 2026-03-27
**Last Reviewed:** 2026-03-27
**Next Review:** 2026-09-27
**Owner:** Engineering Lead, Auramedical Tecnologia
**Classification:** Internal

---

## 1. Purpose

This document provides end-to-end documentation of the account deletion flow in the Aura Medical platform. Account deletion is a mandatory feature required by:

- **Apple App Store Review Guidelines** (Section 5.1.1) — Apps that support account creation must offer account deletion
- **LGPD** (Art. 18, VI) — Right to deletion of personal data processed with consent
- **HIPAA** — While HIPAA does not mandate deletion, it requires that retained data (audit logs) be maintained for compliance periods

This document covers the complete flow from user initiation through final data purge, including all affected data stores and compliance considerations.

## 2. Scope

This procedure affects all data stores in the Aura Medical ecosystem:

- iOS local storage (SwiftData, Keychain)
- Supabase database (PostgreSQL)
- Supabase Auth (user sessions, tokens)
- Backend API state (if any)

---

## 3. iOS Client Flow

### 3.1 User Interface

**Navigation path:** Voce (Profile tab) > Configuracoes (Settings) > "Apagar minha conta"

#### 3.1.1 Step 1: Initiation

The user taps "Apagar minha conta" (Delete my account) in the Settings screen.

#### 3.1.2 Step 2: Confirmation Dialog

A confirmation dialog is presented with the following content (PT-BR):

```
Titulo: Apagar minha conta

Voce tem certeza que deseja apagar sua conta?

Esta acao ira:
- Remover todos os seus dados de saude (pontuacoes, exames, biometria)
- Cancelar qualquer assinatura ativa
- Remover seu acesso ao aplicativo

Seus dados serao mantidos por 30 dias caso voce mude de ideia.
Apos esse periodo, todos os dados serao permanentemente apagados.

Registros de auditoria serao anonimizados e mantidos por obrigacao legal.

[Cancelar]  [Apagar Conta]
```

The "Apagar Conta" button is styled as a destructive action (red text).

#### 3.1.3 Step 3: Re-authentication

Before proceeding, the user must re-authenticate:

- If biometric auth is enabled: Face ID / Touch ID prompt
- If not: Password re-entry

This prevents unauthorized deletion if the device is unlocked.

#### 3.1.4 Step 4: Backend Request

Upon confirmation and re-authentication:

```swift
// AuraBackendClient.swift
func deleteAccount() async throws {
    let response = try await authenticatedRequest(
        method: .DELETE,
        path: "/api/users/me"
    )
    guard response.statusCode == 200 else {
        throw AuraBackendError.deletionFailed
    }
}
```

#### 3.1.5 Step 5: Local Data Cleanup

After successful backend response:

```swift
// AuthViewModel.swift — signOut() flow
func signOut() {
    // 1. Clear all SwiftData containers
    clearSwiftData()

    // 2. Clear Keychain
    AuraKeychain.deleteAll()

    // 3. Clear biometric auth state
    BiometricAuthManager.shared.reset()

    // 4. Sign out of Supabase Auth
    Task {
        try? await supabase.auth.signOut()
    }

    // 5. Reset navigation to login screen
    isAuthenticated = false
}
```

#### 3.1.6 Step 6: Redirect

The user is redirected to the login/onboarding screen. The app is in a clean state.

### 3.2 iOS Data Affected

| Data Store | Action | Timing |
|------------|--------|--------|
| SwiftData — `LocalScore` | Delete all records | Immediate (Step 5) |
| SwiftData — `LocalProfile` | Delete all records | Immediate (Step 5) |
| SwiftData — `LocalBiometrics` | Delete all records | Immediate (Step 5) |
| Keychain — auth tokens | Delete all items | Immediate (Step 5) |
| Keychain — biometric preference | Delete | Immediate (Step 5) |
| Keychain — App Attest key ID | Delete | Immediate (Step 5) |
| UserDefaults | Clear app-specific keys | Immediate (Step 5) |
| HealthKit data | **Not deleted** — owned by iOS, not by Aura | N/A |

**Important:** HealthKit data is owned by the iOS Health app and is not deleted when the user deletes their Aura Medical account. The user can delete HealthKit data independently via the iOS Health app.

---

## 4. Backend Flow

### 4.1 API Endpoint

```
DELETE /api/users/me
Authorization: Bearer <JWT>
```

**Authentication:** Valid Supabase JWT required. The endpoint uses the authenticated user's ID from the JWT — users can only delete their own account.

### 4.2 Request Processing

```typescript
// routes/users.ts
app.delete('/api/users/me', authMiddleware, async (c) => {
  const user = c.get('user'); // From authMiddleware

  // Call Supabase RPC with service client (elevated privileges)
  const { error } = await supabaseService
    .rpc('delete_my_account', { target_user_id: user.id });

  if (error) {
    logger.error({ userId: user.id, error }, 'Account deletion failed');
    return c.json({ error: 'Deletion failed' }, 500);
  }

  logger.info({ userId: user.id }, 'Account deletion initiated');
  return c.json({ message: 'Account scheduled for deletion' }, 200);
});
```

### 4.3 Database RPC: `delete_my_account()`

The `delete_my_account()` function is a PostgreSQL function that executes with `SECURITY DEFINER` privileges to perform cross-table operations.

```sql
CREATE OR REPLACE FUNCTION delete_my_account(target_user_id UUID)
RETURNS void AS $$
BEGIN
  -- Verify the caller matches the target (defense in depth)
  IF auth.uid() IS DISTINCT FROM target_user_id THEN
    RAISE EXCEPTION 'Unauthorized: cannot delete another user account';
  END IF;

  -- 1. Soft-delete user profile
  UPDATE user_profiles
  SET deleted_at = NOW(),
      updated_at = NOW()
  WHERE id = target_user_id;

  -- 2. Anonymize audit logs (retain for compliance, remove PII linkage)
  UPDATE ai_audit_log
  SET user_id = '00000000-0000-0000-0000-000000000000'::UUID
  WHERE user_id = target_user_id;

  -- 3. Delete health data (immediate, no hold period)
  DELETE FROM daily_phenomic_scores WHERE user_id = target_user_id;
  DELETE FROM biomarker_results WHERE user_id = target_user_id;
  DELETE FROM healthkit_summaries WHERE user_id = target_user_id;
  DELETE FROM questionnaire_responses WHERE user_id = target_user_id;

  -- 4. Delete communication data
  DELETE FROM chat_messages WHERE user_id = target_user_id;
  DELETE FROM push_tokens WHERE user_id = target_user_id;

  -- 5. Delete active sessions (force sign-out on all devices)
  DELETE FROM auth.sessions WHERE user_id = target_user_id;
  DELETE FROM auth.refresh_tokens WHERE user_id = target_user_id;

  -- 6. Log the deletion event (with anonymized reference)
  INSERT INTO ai_audit_log (
    user_id,
    operation,
    engine_version,
    input_hash,
    created_at
  ) VALUES (
    '00000000-0000-0000-0000-000000000000'::UUID,
    'account_deletion',
    'system',
    encode(sha256(target_user_id::text::bytea), 'hex'),
    NOW()
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### 4.4 Deferred Hard-Delete (30-Day Cron)

A cron job runs daily to permanently delete accounts past the 30-day hold period:

```sql
-- Scheduled via pg_cron: daily at 03:00 UTC
CREATE OR REPLACE FUNCTION hard_delete_expired_accounts()
RETURNS void AS $$
DECLARE
  deleted_count INTEGER;
BEGIN
  -- Delete user profiles past 30-day hold
  WITH deleted AS (
    DELETE FROM user_profiles
    WHERE deleted_at IS NOT NULL
      AND deleted_at < NOW() - INTERVAL '30 days'
    RETURNING id
  )
  SELECT COUNT(*) INTO deleted_count FROM deleted;

  -- Log the operation
  IF deleted_count > 0 THEN
    INSERT INTO ai_audit_log (
      user_id,
      operation,
      engine_version,
      input_hash,
      created_at
    ) VALUES (
      '00000000-0000-0000-0000-000000000000'::UUID,
      'hard_delete_expired_accounts',
      'system',
      encode(sha256(deleted_count::text::bytea), 'hex'),
      NOW()
    );
  END IF;

  -- Delete the Supabase Auth user record
  -- Note: This permanently removes the user from auth.users
  -- Must be done after profile deletion to avoid FK issues
  DELETE FROM auth.users
  WHERE id IN (
    SELECT id FROM auth.users
    WHERE id NOT IN (SELECT id FROM user_profiles)
    AND created_at < NOW() - INTERVAL '30 days'
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Schedule
SELECT cron.schedule(
  'hard-delete-expired-accounts',
  '0 3 * * *',
  'SELECT hard_delete_expired_accounts()'
);
```

---

## 5. Complete Data Lifecycle

### 5.1 Data Affected by Account Deletion

| Data Type | Database Table | Action at Deletion | Hold Period | Final State |
|-----------|---------------|-------------------|-------------|-------------|
| User profile | `user_profiles` | Soft-delete (`deleted_at = NOW()`) | 30 days | Hard-deleted |
| Auth user record | `auth.users` | Sessions deleted immediately | 30 days (tied to profile) | Hard-deleted |
| Auth sessions | `auth.sessions` | Deleted immediately | None | Gone |
| Auth refresh tokens | `auth.refresh_tokens` | Deleted immediately | None | Gone |
| Daily phenomic scores | `daily_phenomic_scores` | Deleted immediately | None | Gone |
| Lab results | `biomarker_results` | Deleted immediately | None | Gone |
| HealthKit summaries | `healthkit_summaries` | Deleted immediately | None | Gone |
| Questionnaire responses | `questionnaire_responses` | Deleted immediately | None | Gone |
| Chat messages | `chat_messages` | Deleted immediately | None | Gone |
| Push notification tokens | `push_tokens` | Deleted immediately | None | Gone |
| Audit logs (SaMD) | `ai_audit_log` | Anonymized (`user_id` = null UUID) | **7 years** (anonymized) | Retained anonymized |
| Order/prescription data | `orders` | Anonymized | **5 years** (legal req.) | Retained anonymized |
| Local SwiftData | Device | Deleted immediately | None | Gone |
| Keychain data | Device | Deleted immediately | None | Gone |

### 5.2 Data NOT Deleted

| Data | Reason | Legal Basis |
|------|--------|-------------|
| Anonymized audit logs | HIPAA 164.530(j) — 6-year retention; CFR 21 Part 11 — SaMD records | LGPD Art. 16, I — legal obligation |
| Anonymized order data | Brazilian healthcare record-keeping requirements | LGPD Art. 16, I — legal obligation |
| Aggregated analytics | Already de-identified; no PII linkage | LGPD Art. 12 — anonymized data not subject to LGPD |
| HealthKit data on device | Owned by iOS Health app, not Aura Medical | Outside Aura's data controller scope |

### 5.3 Deletion Timeline

```
Day 0:  User requests account deletion
        ├── iOS: Local data cleared immediately (SwiftData + Keychain)
        ├── Backend: Health data deleted immediately
        ├── Backend: User profile soft-deleted
        ├── Backend: Audit logs anonymized
        └── Backend: Sessions terminated

Day 1-30: Grace period
        ├── User can contact support to reverse deletion
        ├── Profile exists with deleted_at set
        └── No health data remains (already deleted)

Day 30: Hard-delete cron runs
        ├── User profile permanently deleted
        ├── Auth user record deleted
        └── Deletion logged in audit trail
```

---

## 6. Account Recovery (Grace Period)

### 6.1 Recovery Process

During the 30-day hold period, a user may request account recovery:

1. User contacts support at suporte@auramedical.com
2. Identity verification via email associated with the account
3. Support executes account restoration:

```sql
-- Restore soft-deleted account
UPDATE user_profiles
SET deleted_at = NULL,
    updated_at = NOW()
WHERE id = [user_id]
  AND deleted_at IS NOT NULL
  AND deleted_at > NOW() - INTERVAL '30 days';
```

4. User can sign in again and re-sync data from HealthKit
5. **Note:** Health data (scores, labs, biometrics) deleted at Step 0 is NOT recoverable

### 6.2 Limitations

- Health scores, lab results, biometrics, questionnaire responses, and chat messages are deleted immediately and cannot be recovered
- Only the user profile (name, email, preferences) is recoverable during the grace period
- The user must re-sync HealthKit data and rebuild their health history
- Audit logs remain anonymized even if the account is restored

---

## 7. Compliance Verification

### 7.1 Apple App Store Compliance

| Requirement | Implementation | Status |
|-------------|---------------|--------|
| Account deletion must be available in-app | Settings > "Apagar minha conta" | Implemented |
| Deletion must be easy to find | One level deep in Settings | Implemented |
| Must delete associated data | All health data deleted immediately; profile within 30 days | Implemented |
| Must cancel subscriptions | Included in deletion flow (future: StoreKit integration) | Planned |
| Must work without contacting support | Fully self-service in-app | Implemented |

### 7.2 LGPD Compliance

| Requirement | Article | Implementation | Status |
|-------------|---------|---------------|--------|
| Right to deletion | Art. 18, VI | Complete deletion flow with 30-day grace | Implemented |
| Data controller must respond within 15 days | Art. 18, Par. 5 | Immediate deletion (exceeds requirement) | Implemented |
| Retention only with legal basis | Art. 16 | Audit logs retained per legal obligation (Art. 16, I) | Implemented |
| Inform user about deletion consequences | Art. 18, Par. 1 | Confirmation dialog explains impact | Implemented |
| Right to revocation of consent | Art. 18, IX | Deletion effectively revokes all consent | Implemented |

### 7.3 HIPAA Compliance

| Requirement | Regulation | Implementation | Status |
|-------------|-----------|---------------|--------|
| Retain documentation for 6 years | 164.530(j) | Audit logs anonymized and retained 7 years | Implemented |
| Accounting of disclosures | 164.528 | Audit log entries preserved (anonymized) | Implemented |
| PHI disposal | 164.310(d)(2)(i) | Health data hard-deleted; no remnants | Implemented |

---

## 8. Testing and Verification

### 8.1 Test Cases

| Test Case | Expected Result | Verified |
|-----------|----------------|----------|
| User taps "Apagar minha conta" | Confirmation dialog appears | |
| User confirms deletion | Backend receives DELETE request | |
| After deletion: local SwiftData | All LocalScore, LocalProfile, LocalBiometrics records deleted | |
| After deletion: Keychain | All Aura entries removed | |
| After deletion: user redirected | Login/onboarding screen shown | |
| After deletion: daily_phenomic_scores | All user's records deleted | |
| After deletion: biomarker_results | All user's records deleted | |
| After deletion: ai_audit_log | user_id set to null UUID | |
| After deletion: auth.sessions | All user's sessions deleted | |
| After 30 days: user_profiles | Profile hard-deleted by cron | |
| Recovery within 30 days | Profile restored, health data NOT recovered | |
| Re-authentication required | Biometric/password prompt before deletion | |

### 8.2 Monitoring

- Monitor `ai_audit_log` for `account_deletion` operations
- Monitor `hard_delete_expired_accounts` cron job success/failure
- Alert on deletion failures (HTTP 500 from DELETE /api/users/me)
- Quarterly audit: verify no orphaned data exists for deleted accounts

---

## 9. Error Handling

### 9.1 Client-Side Errors

| Error | User-Facing Message (PT-BR) | Recovery |
|-------|------------------------------|----------|
| Network failure | "Nao foi possivel conectar ao servidor. Tente novamente." | Retry button |
| Backend 500 | "Erro ao processar sua solicitacao. Tente novamente mais tarde." | Retry later |
| Auth failure (re-auth) | "Autenticacao falhou. Verifique suas credenciais." | Re-enter credentials |
| Backend timeout | "O servidor demorou para responder. Tente novamente." | Retry button |

### 9.2 Server-Side Errors

| Error | Handling | Alert |
|-------|----------|-------|
| RPC failure | Return 500, log error with user_id | Immediate alert to engineering |
| Partial deletion (some tables fail) | Transaction rollback; return 500 | P1 alert — data consistency issue |
| Cron job failure | Log error; retry on next run | Alert if fails 3 consecutive days |

### 9.3 Data Consistency

The `delete_my_account()` RPC runs as a single transaction. If any step fails:

- The entire transaction is rolled back
- The user's account remains active
- The error is logged
- The user receives an error message and can retry

---

## 10. Sequence Diagram

```
User                    iOS App                 Backend API            Supabase DB
 |                        |                        |                      |
 |-- Tap "Apagar" ------->|                        |                      |
 |                        |-- Show confirmation --->|                      |
 |-- Confirm ------------>|                        |                      |
 |                        |-- Re-auth prompt ------>|                      |
 |-- Authenticate ------->|                        |                      |
 |                        |                        |                      |
 |                        |-- DELETE /api/users/me ->|                     |
 |                        |                        |-- RPC delete_my_account()
 |                        |                        |                      |
 |                        |                        |    Soft-delete profile |
 |                        |                        |    Anonymize audits   |
 |                        |                        |    Delete health data |
 |                        |                        |    Delete sessions    |
 |                        |                        |                      |
 |                        |                        |<-- Success -----------|
 |                        |<-- 200 OK --------------|                      |
 |                        |                        |                      |
 |                        |-- Clear SwiftData       |                      |
 |                        |-- Clear Keychain        |                      |
 |                        |-- Sign out Supabase     |                      |
 |                        |                        |                      |
 |<-- Login screen -------|                        |                      |
 |                        |                        |                      |
 |                        |            [30 days later - cron]              |
 |                        |                        |-- hard_delete_expired |
 |                        |                        |    Delete profile     |
 |                        |                        |    Delete auth.users  |
```

---

## 11. Related Documents

- [DATA_RETENTION.md](DATA_RETENTION.md) — Retention periods for each data type
- [BREACH_RESPONSE.md](BREACH_RESPONSE.md) — Breach procedures (relevant if deletion fails and data is exposed)
- [BAA_STATUS.md](BAA_STATUS.md) — Vendor data handling obligations upon account deletion
- `AuraMedical/Security/AuraKeychain.swift` — Keychain cleanup implementation
- `AuraMedical/Security/BiometricAuthManager.swift` — Biometric state reset
- `AuraMedical/LocalData/SyncManager.swift` — Local data management
- `AuraMedical/Services/AuraBackendClient.swift` — Backend API client

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-27 | Engineering Lead | Initial version |
