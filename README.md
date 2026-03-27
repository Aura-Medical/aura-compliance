# Aura Medical — Compliance & Regulatory Documentation

Centralized compliance documentation for the Aura Medical platform.
Covers HIPAA, LGPD, IEC 62304 (SaMD Class B), and cross-repo governance.

## Structure

```
samd/                  # IEC 62304 Class B — Software as Medical Device
  INTENDED_USE.md      # Product classification, intended use, disclaimers
  RISK_ANALYSIS.md     # ISO 14971 FMEA (20 hazards, RPN calculations)
  TRACEABILITY.md      # Requirements → Design → Code → Test matrix
  VERIFICATION.md      # Test strategy, coverage targets, release criteria
  CONFIG_MGMT.md       # 3-repo sync protocol, versioning, change control

hipaa/                 # HIPAA + LGPD administrative safeguards
  BREACH_RESPONSE.md   # Incident response SOP (LGPD 72h notification)
  DATA_RETENTION.md    # Retention periods with legal basis
  BAA_STATUS.md        # Business Associate Agreement vendor tracker
  ACCOUNT_DELETION.md  # End-to-end account deletion flow

cross-repo/            # Cross-repository governance
  SYNC_PROTOCOL.md     # Engine sync rules across 3 repos
  DATA_CONTRACT.md     # Shared types: PhenomicInput, PhenomicResult, /engine/validate
```

## Related Repositories

| Repo | Purpose |
|------|---------|
| [phenomic-engine](https://github.com/Aura-Medical/phenomic-engine) | Licensable scoring engine (TypeScript) |
| [aura-backend](https://github.com/Aura-Medical/aura-backend) | Health API: engine validation, data sync |
| [aura-ios](https://github.com/Aura-Medical/aura-ios) | iOS app (SwiftUI) |

## SaMD Classification

- **Class:** IEC 62304 Class B / IMDRF Category II
- **Purpose:** Informational wellness scoring (allostatic load, 0-100)
- **NOT:** Diagnostic, prescriptive, or treatment-controlling

## Compliance Status

See `hipaa/BAA_STATUS.md` for vendor BAA tracking and `samd/INTENDED_USE.md` for regulatory classification details.
