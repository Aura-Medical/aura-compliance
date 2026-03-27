# Breach Response Standard Operating Procedure

**Document ID:** AURA-SOP-BR-001
**Version:** 1.0
**Effective Date:** 2026-03-27
**Last Reviewed:** 2026-03-27
**Next Review:** 2026-09-27
**Owner:** Security Officer, Auramedical Tecnologia
**Classification:** Internal — Confidential

---

## 1. Purpose

This Standard Operating Procedure (SOP) defines the process for detecting, containing, assessing, notifying, and remediating a security breach involving Protected Health Information (PHI) or Personally Identifiable Information (PII) within the Aura Medical platform. This procedure ensures compliance with:

- **HIPAA Breach Notification Rule** (45 CFR 164.400-414)
- **LGPD** (Lei Geral de Protecao de Dados, Lei 13.709/2018) — Articles 46-49
- **Apple App Store Guidelines** — Privacy and data handling requirements

## 2. Scope

This SOP applies to all components of the Aura Medical platform:

- **iOS Application** (AuraMedical)
- **Backend API** (Hono on Render)
- **Database** (Supabase — PostgreSQL)
- **Third-party integrations** (HealthKit, authentication providers)
- **All team members** with access to production systems

## 3. Definitions

| Term | Definition |
|------|-----------|
| **Breach** | Unauthorized acquisition, access, use, or disclosure of PHI that compromises the security or privacy of such information |
| **PHI** | Protected Health Information — individually identifiable health information (lab results, biometrics, health scores, questionnaire responses) |
| **PII** | Personally Identifiable Information — name, email, date of birth, etc. |
| **Incident** | A security event that may or may not constitute a breach; requires investigation |
| **Breach Response Team** | Security Officer, CTO, Engineering Lead, Legal Counsel |

## 4. Severity Classification

| Level | Description | Examples | Response Time |
|-------|-------------|----------|---------------|
| **P0 — Critical** | Confirmed PHI breach affecting multiple users | Database exfiltration, compromised service key | Immediate (< 1 hour) |
| **P1 — High** | Confirmed unauthorized access to PHI, limited scope | Single account compromise, insider access | < 4 hours |
| **P2 — Medium** | Potential PHI exposure, no confirmed access | Misconfigured RLS policy, exposed endpoint | < 24 hours |
| **P3 — Low** | Security incident with no PHI involvement | Failed auth attempts, non-PHI data leak | < 72 hours |

---

## 5. Phase 1: Detection

### 5.1 Automated Detection Sources

#### 5.1.1 Audit Log Anomalies (`ai_audit_log`)

The `ai_audit_log` table records all SaMD engine operations with SHA-256 input hashes. Monitor for:

- **Unexpected operation types** — Operations outside normal `score_calculation`, `input_validation`, `clinical_override` patterns
- **Bulk access patterns** — Unusual volume of queries from a single `user_id` or `session_id` (threshold: >100 operations/hour per user)
- **Off-hours activity** — Engine operations outside expected usage patterns (configurable by timezone)
- **Missing or invalid input hashes** — Indicates potential log tampering

```sql
-- Example: Detect anomalous bulk access in the last 24 hours
SELECT user_id, COUNT(*) AS op_count, MIN(created_at) AS first_op, MAX(created_at) AS last_op
FROM ai_audit_log
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY user_id
HAVING COUNT(*) > 100
ORDER BY op_count DESC;
```

#### 5.1.2 Authentication Failure Monitoring

- **Brute force detection** — More than 10 failed login attempts from a single IP within 15 minutes
- **Credential stuffing** — High volume of failed logins across multiple accounts from similar IP ranges
- **Token anomalies** — JWT validation failures, expired token reuse attempts, tokens from revoked sessions
- **App Attest failures** — Invalid attestation from purported iOS clients (indicates API abuse or client tampering)

```sql
-- Example: Detect brute force attempts via Supabase auth logs
SELECT raw_user_meta_data->>'ip' AS ip_address, COUNT(*) AS failures
FROM auth.audit_log_entries
WHERE action = 'login' AND payload->>'success' = 'false'
  AND created_at > NOW() - INTERVAL '15 minutes'
GROUP BY raw_user_meta_data->>'ip'
HAVING COUNT(*) > 10;
```

#### 5.1.3 Supabase Platform Alerts

- **Database alerts** — Unusual query patterns detected by pgaudit
- **Storage alerts** — Unauthorized access attempts to storage buckets
- **Edge Function errors** — Unexpected error rates or response patterns
- **Connection pool exhaustion** — May indicate a denial-of-service or data exfiltration attempt

#### 5.1.4 Render (Backend) Monitoring

- **Error rate spikes** — Sudden increase in 4xx/5xx responses on PHI endpoints
- **Latency anomalies** — Unexpected response time changes (may indicate data exfiltration)
- **Deployment anomalies** — Unauthorized or unexpected deployments
- **Outbound traffic spikes** — Unusual egress from the backend service

### 5.2 Manual Detection Sources

#### 5.2.1 User Reports

- In-app feedback mechanism
- Support email: security@auramedical.com
- All user reports of suspicious account activity must be escalated within 4 hours

#### 5.2.2 Team Member Reports

- Any team member who observes suspicious activity must immediately notify the Security Officer
- No retaliation for good-faith security reports

#### 5.2.3 External Reports

- Responsible disclosure program
- Vendor security notifications (Supabase, Render, Apple)
- Threat intelligence feeds

### 5.3 Detection Escalation

Upon detection of a potential incident:

1. **Log the incident** — Create an entry in the incident tracking system with timestamp, source, and initial description
2. **Classify severity** — Use the severity table in Section 4
3. **Notify Breach Response Team** — Via secure communication channel (not email for P0/P1)
4. **Begin Phase 2** — Proceed to containment immediately for P0/P1; within assessment timeline for P2/P3

---

## 6. Phase 2: Containment

### 6.1 Immediate Containment Actions (P0/P1)

Execute these actions in order. Each action should be logged with timestamp and operator.

#### 6.1.1 Revoke Supabase Service Role Key

1. Navigate to Supabase Dashboard > Project Settings > API
2. Click "Regenerate" on the service_role key
3. **Impact:** All backend operations using the service client will fail until the new key is deployed
4. Deploy the new key to Render environment variables immediately

```bash
# On Render Dashboard or CLI:
# Update SUPABASE_SERVICE_ROLE_KEY environment variable
# Trigger redeploy
```

#### 6.1.2 Disable Compromised Endpoints

1. If a specific endpoint is compromised, deploy a hotfix to return 503 on that route
2. If the entire backend is compromised:
   - Suspend the Render service
   - Enable Supabase RLS on all tables (verify policies are active)
   - Disable the Supabase REST API if direct database access is suspected

#### 6.1.3 Rotate JWT Secrets

1. Navigate to Supabase Dashboard > Project Settings > API
2. Regenerate the JWT secret
3. **Impact:** All existing user sessions become invalid — users must re-authenticate
4. Update the JWT secret in backend environment variables
5. Update `SUPABASE_JWT_SECRET` in Render

#### 6.1.4 Force Sign-Out All Sessions

```sql
-- Terminate all active sessions in Supabase Auth
DELETE FROM auth.sessions;
DELETE FROM auth.refresh_tokens;
```

#### 6.1.5 Invalidate App Attest Keys (if client compromise suspected)

1. Revoke the App Attest assertion key stored in Supabase
2. Force re-attestation on next app launch
3. Update the `AppAttestManager` server-side validation endpoint

### 6.2 Network Containment

- Block suspicious IP ranges at the CDN/WAF level
- Enable additional rate limiting on all PHI endpoints
- Enable enhanced logging on all API routes

### 6.3 Preservation of Evidence

- **Do NOT delete logs** — All `ai_audit_log`, `pgaudit`, and application logs must be preserved
- Take database snapshots before any remediation
- Export relevant Render logs to secure storage
- Document all containment actions with exact timestamps

---

## 7. Phase 3: Assessment

### 7.1 Determine Scope of Access

#### 7.1.1 Query Audit Logs

```sql
-- Determine what data was accessed during the breach window
SELECT
  user_id,
  operation,
  input_hash,
  engine_version,
  created_at
FROM ai_audit_log
WHERE created_at BETWEEN '[breach_start]' AND '[breach_end]'
ORDER BY created_at;
```

#### 7.1.2 Query Data Access Patterns

```sql
-- Identify affected users by checking data access during breach window
SELECT DISTINCT user_id
FROM daily_phenomic_scores
WHERE updated_at BETWEEN '[breach_start]' AND '[breach_end]';

-- Check biomarker result access
SELECT DISTINCT user_id
FROM biomarker_results
WHERE updated_at BETWEEN '[breach_start]' AND '[breach_end]';
```

#### 7.1.3 Supabase Auth Audit

```sql
-- Review authentication events during breach window
SELECT *
FROM auth.audit_log_entries
WHERE created_at BETWEEN '[breach_start]' AND '[breach_end]'
ORDER BY created_at;
```

### 7.2 Count Affected Users

Document the exact number of affected users, broken down by:

- Users whose PHI was confirmed accessed
- Users whose PHI was potentially accessed (same dataset, access uncertain)
- Users whose non-PHI data was accessed

### 7.3 Determine Exposure Period

- **Start time:** Earliest evidence of unauthorized access
- **End time:** When containment was completed
- **Duration:** Calculate total exposure window

### 7.4 Classify Data Exposed

| Data Category | PHI? | Impact Level |
|---------------|------|-------------|
| Health scores (daily_phenomic_scores) | Yes | High |
| Lab results (biomarker_results) | Yes | High |
| Biometric data (HealthKit summaries) | Yes | High |
| Questionnaire responses | Yes | Medium |
| User profiles (name, email, DOB) | PII | Medium |
| Audit logs (ai_audit_log) | Contains PHI references | Medium |
| Authentication tokens | No (but enables access) | High |
| App configuration | No | Low |

### 7.5 Risk Assessment

Apply the HIPAA four-factor risk assessment:

1. **Nature and extent of PHI involved** — Types of identifiers and clinical data exposed
2. **Unauthorized person who accessed the PHI** — Known or unknown threat actor, insider vs external
3. **Whether PHI was actually acquired or viewed** — Evidence of data exfiltration vs incidental access
4. **Extent of risk mitigation** — Effectiveness of containment actions

**Determination:** If the risk assessment concludes there is greater than a low probability that PHI was compromised, it constitutes a reportable breach.

---

## 8. Phase 4: Notification

### 8.1 LGPD Notification (Brazilian Law — Primary Jurisdiction)

**Timeline:** Communication to ANPD within **reasonable time** (ANPD Regulation recommends 72 hours for incidents of relevant risk — Resolucao CD/ANPD No. 15/2024)

#### 8.1.1 Notify ANPD (Autoridade Nacional de Protecao de Dados)

Submit incident report via ANPD's Peticionamento Eletronico system:
- URL: https://www.gov.br/anpd/pt-br
- Include: nature of data, affected data subjects count, technical measures adopted, risks, measures to mitigate

#### 8.1.2 Notify Affected Users (PT-BR Template)

```
Assunto: Aviso Importante de Seguranca — Aura Medical

Prezado(a) [NOME],

Estamos escrevendo para informa-lo(a) sobre um incidente de seguranca que
pode ter afetado seus dados no aplicativo Aura Medical.

O QUE ACONTECEU:
Em [DATA], identificamos um acesso nao autorizado aos nossos sistemas que
pode ter resultado na exposicao de [TIPO DE DADOS AFETADOS].

QUAIS DADOS FORAM AFETADOS:
[Lista especifica dos tipos de dados — ex: pontuacoes de saude, resultados
de exames laboratoriais, dados biometricos]

O QUE ESTAMOS FAZENDO:
- Contivemos o incidente imediatamente apos a deteccao
- Realizamos uma investigacao completa
- Implementamos medidas de seguranca adicionais
- Notificamos a Autoridade Nacional de Protecao de Dados (ANPD)

O QUE VOCE PODE FAZER:
1. Altere sua senha do Aura Medical imediatamente
2. Se voce usa a mesma senha em outros servicos, altere-as tambem
3. Monitore suas contas para atividades suspeitas
4. Entre em contato conosco se notar algo incomum

SEUS DIREITOS (LGPD Art. 18):
Voce tem direito a:
- Confirmacao da existencia de tratamento de dados
- Acesso aos seus dados
- Correcao de dados incompletos ou desatualizados
- Anonimizacao, bloqueio ou eliminacao de dados
- Portabilidade dos dados
- Eliminacao dos dados pessoais tratados com consentimento
- Revogacao do consentimento

Para exercer qualquer desses direitos ou para mais informacoes, entre em
contato conosco:

Email: privacidade@auramedical.com
Encarregado de Dados (DPO): [NOME DO DPO]

Pedimos sinceras desculpas por este incidente e estamos comprometidos em
proteger seus dados.

Atenciosamente,
Equipe Aura Medical
Auramedical Tecnologia
```

### 8.2 HIPAA Notification (If Applicable)

HIPAA applies if Aura Medical processes PHI of individuals covered by U.S. health plans or if Aura Medical operates as a covered entity or business associate under U.S. law.

**Timeline:** Notification to HHS within **60 calendar days** of discovery.

#### 8.2.1 HHS Notification

- **Breaches affecting 500+ individuals:** Notify HHS immediately via the HHS Breach Portal (https://ocrportal.hhs.gov/ocr/breach/wizard_breach.jsf)
- **Breaches affecting <500 individuals:** Notify HHS within 60 days of end of calendar year in which breach was discovered
- **Media notification** (500+ in a single state/jurisdiction): Notify prominent media outlets

#### 8.2.2 Individual Notification (English Template)

```
Subject: Important Security Notice — Aura Medical

Dear [NAME],

We are writing to inform you about a security incident that may have
affected your data in the Aura Medical application.

WHAT HAPPENED:
On [DATE], we identified unauthorized access to our systems that may have
resulted in the exposure of [TYPES OF DATA AFFECTED].

WHAT INFORMATION WAS INVOLVED:
[Specific list of data types — e.g., health scores, laboratory results,
biometric data]

WHAT WE ARE DOING:
- We contained the incident immediately upon detection
- We conducted a thorough investigation
- We implemented additional security measures
- We have notified relevant regulatory authorities

WHAT YOU CAN DO:
1. Change your Aura Medical password immediately
2. If you use the same password on other services, change those as well
3. Monitor your accounts for suspicious activity
4. Contact us if you notice anything unusual

For more information or to exercise your privacy rights, contact us:

Email: privacy@auramedical.com
Privacy Officer: [NAME]

We sincerely apologize for this incident and are committed to protecting
your data.

Sincerely,
Aura Medical Team
Auramedical Tecnologia
```

### 8.3 Notification Log

Maintain a log of all notifications sent:

| Recipient | Method | Date/Time | Content Reference | Confirmation |
|-----------|--------|-----------|-------------------|-------------|
| ANPD | Electronic petition | | | |
| HHS (if applicable) | Breach portal | | | |
| Affected users | Email | | Template v[X] | |
| Media (if required) | Press release | | | |

---

## 9. Phase 5: Remediation

### 9.1 Root Cause Analysis

Conduct a formal root cause analysis (RCA) within 7 days of containment:

1. **Timeline reconstruction** — Minute-by-minute account of the incident
2. **Attack vector identification** — How the breach occurred
3. **Contributing factors** — System weaknesses that enabled the breach
4. **Root cause** — The fundamental cause (e.g., missing input validation, misconfigured RLS, compromised credential)

### 9.2 Patch Deployment

1. Develop and test the fix in a staging environment
2. Conduct security review of the patch
3. Deploy to production with enhanced monitoring
4. Verify the fix resolves the vulnerability

### 9.3 Re-Audit

After remediation, conduct:

- Full security audit of the affected component
- Review of all RLS policies in Supabase
- Review of all API endpoint authorization
- Penetration testing of the remediated area
- Verification that `ai_audit_log` integrity is maintained

### 9.4 Post-Mortem Document

Produce a written post-mortem within 14 days containing:

- Incident summary
- Timeline of events
- Root cause analysis
- Impact assessment (users affected, data exposed, financial impact)
- Remediation actions taken
- Lessons learned
- Action items with owners and deadlines

---

## 10. Phase 6: Post-Incident

### 10.1 Update Security Controls

Based on lessons learned:

- Update certificate pinning configuration if TLS was compromised
- Strengthen RLS policies if database access was exploited
- Enhance rate limiting if brute force was involved
- Add additional audit log fields if detection was delayed
- Review and update `PHIProtectionModifier` coverage in iOS app

### 10.2 Team Training

- Conduct security awareness training within 30 days of the incident
- Review this SOP with all team members
- Conduct tabletop exercise simulating a similar breach

### 10.3 Update Documentation

- Update this SOP based on lessons learned
- Update [DATA_RETENTION.md](DATA_RETENTION.md) if retention policies change
- Update [BAA_STATUS.md](BAA_STATUS.md) if vendor relationships are affected
- Update [ACCOUNT_DELETION.md](ACCOUNT_DELETION.md) if deletion procedures change

### 10.4 Regulatory Follow-Up

- Respond to any ANPD inquiries
- Respond to any HHS inquiries (if applicable)
- File supplemental reports if new information emerges
- Close the incident when all action items are complete

---

## 11. Roles and Responsibilities

| Role | Responsibility | Contact |
|------|---------------|---------|
| **Security Officer** | Overall incident coordination, ANPD/HHS communication | TBD |
| **CTO** | Technical decisions, containment authorization | TBD |
| **Engineering Lead** | Technical investigation, patch development | TBD |
| **Legal Counsel** | Regulatory compliance, notification review | TBD |
| **DPO (Encarregado)** | Data subject rights, LGPD compliance | TBD |

## 12. Communication Channels

| Severity | Primary Channel | Backup Channel |
|----------|----------------|----------------|
| P0/P1 | Secure messaging (Signal) | Phone call |
| P2/P3 | Encrypted email | Secure messaging |
| Post-incident | Email + Document | Meeting |

**Rule:** Never discuss breach details over unencrypted email or public Slack channels.

## 13. Testing and Maintenance

- **Tabletop exercises:** Conduct at least annually, simulating a P1 breach
- **SOP review:** Every 6 months or after any incident
- **Contact verification:** Quarterly verification that all contact information is current
- **Detection testing:** Monthly verification that anomaly detection queries are functioning

---

## 14. Related Documents

- [DATA_RETENTION.md](DATA_RETENTION.md) — Data retention policies and purge schedules
- [BAA_STATUS.md](BAA_STATUS.md) — Business Associate Agreement status with vendors
- [ACCOUNT_DELETION.md](ACCOUNT_DELETION.md) — Account deletion procedures
- `AuraMedical/Security/EngineAuditLogger.swift` — SaMD audit trail implementation
- `AuraMedical/Security/CertificatePinning.swift` — Certificate pinning configuration
- `AuraMedical/Security/AuraKeychain.swift` — Keychain storage implementation

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-03-27 | Security Officer | Initial version |
