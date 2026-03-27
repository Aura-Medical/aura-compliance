# Configuration Management Plan

**Document ID:** CM-001
**Revision:** 1.0
**Date:** 2026-03-27
**Product:** Aura Medical iOS Application
**Standard:** IEC 62304:2006+AMD1:2015 Section 5.1.9 (Software configuration management)

---

## 1. Purpose

This document defines the configuration management strategy for the Aura Medical software system, including version control, the 3-repository synchronization protocol, change control procedures, build processes, and release management.

## 2. Repository Architecture

The Aura Medical system spans three repositories that must remain synchronized for SaMD compliance:

```
aura-wearables/
  |-- phenomic-engine/     # TypeScript canonical engine (source of truth)
  |-- aura-backend/        # Hono backend (consumes TS engine)
  |-- aura-ios/            # Swift iOS app (ports TS engine)
```

### 2.1 Repository Roles

| Repository        | Language   | Role                           | Engine Path                   |
|-------------------|------------|--------------------------------|-------------------------------|
| `phenomic-engine` | TypeScript | **Canonical** engine source     | `src/engine.ts`, `src/config.ts` |
| `aura-backend`    | TypeScript | Backend consumer + cross-validator | Imports from phenomic-engine  |
| `aura-ios`        | Swift      | iOS port of engine              | `Engine/PhenomicEngine.swift`, `Engine/EngineConfig.swift` |

### 2.2 Source of Truth Hierarchy

1. **Engine algorithm:** `phenomic-engine/` TypeScript implementation is the canonical source.
2. **Test vectors:** `phenomic-engine/parity-vectors.json` defines expected outputs for canonical inputs.
3. **Configuration:** `phenomic-engine/src/config.ts` defines all default thresholds, weights, and ranges.
4. **Swift port:** `aura-ios/Engine/` must match the TypeScript implementation exactly (integer output parity).

## 3. Version Scheme

### 3.1 Engine Version

The engine uses semantic versioning with a platform suffix:

| Platform   | Format          | Current Version  | Example       |
|------------|-----------------|------------------|---------------|
| TypeScript | `X.Y.Z`        | `4.2.0`          | `4.2.0`       |
| Swift      | `X.Y.Z-swift`  | `4.2.0-swift`    | `4.2.0-swift` |

**Version Rules:**
- **Major (X):** Breaking changes to input/output schema or scoring algorithm fundamentals.
- **Minor (Y):** New features (e.g., new biomarker support, new domain), backward-compatible.
- **Patch (Z):** Bug fixes, threshold adjustments, config changes that don't alter behavior for existing inputs.

Both platforms **must** share the same `X.Y.Z` base version. The `-swift` suffix distinguishes the port but confirms algorithmic parity.

### 3.2 Application Version

| Component   | Versioning    | Current  | Source                              |
|-------------|---------------|----------|-------------------------------------|
| iOS App     | SemVer        | 1.0.0    | `project.yml` / Xcode build settings |
| Backend API | SemVer        | 1.0.0    | `package.json`                       |
| Engine      | SemVer+suffix | 4.2.0    | `PhenomicEngine.engineVersion`       |

### 3.3 Version Tracking in Code

```swift
// Engine/PhenomicEngine.swift
struct PhenomicEngine {
    static let engineVersion = "4.2.0-swift"
    // ...
}
```

```typescript
// phenomic-engine/src/engine.ts
export const ENGINE_VERSION = "4.2.0";
```

## 4. Three-Repository Sync Protocol

### 4.1 Change Flow (CRITICAL)

Any change to engine math, thresholds, weights, or scoring logic **MUST** follow this protocol:

```
Step 1: Implement in phenomic-engine/ (TypeScript)
         |
Step 2: Update/add parity test vectors in parity-vectors.json
         |
Step 3: Run vitest -- all vectors must pass
         |
Step 4: Port changes to aura-ios/Engine/ (Swift)
         |
Step 5: Run XCTest parity tests -- all vectors must pass
         |
Step 6: Update engine version in BOTH repos (same X.Y.Z)
         |
Step 7: Update aura-backend/ if API contract changes
         |
Step 8: Cross-validation integration test
         |
Step 9: Merge PRs in all affected repos
```

### 4.2 Sync Verification Checklist

Before merging any engine change PR, verify:

- [ ] TypeScript parity tests pass (vitest)
- [ ] Swift parity tests pass (XCTest)
- [ ] Engine version bumped in both repos
- [ ] `parity-vectors.json` updated if behavior changes
- [ ] Cross-validation delta <= 2 for all test vectors
- [ ] `EngineConfig.default` updated in both repos if config changes
- [ ] CHANGELOG.md updated in both repos
- [ ] PR description references the sync protocol

### 4.3 Config-Only Changes

Changes to `EngineConfig` that do not alter scoring logic (e.g., adjusting a threshold):

1. Update `phenomic-engine/src/config.ts`.
2. Update `aura-ios/Engine/EngineConfig.swift` `.default` static property.
3. Run parity tests on both platforms.
4. Optionally update remote config in Supabase `engine_config` table.

### 4.4 Drift Detection

The system includes automated drift detection:

- **Runtime:** Every iOS engine computation is cross-validated against the TypeScript engine via `POST /api/engine/validate`. Delta > 2 triggers a warning log.
- **CI:** Parity test vectors run in both TypeScript (vitest) and Swift (XCTest) CI pipelines.
- **Manual:** Monthly audit of engine version parity across all three repos.

## 5. Change Control

### 5.1 Change Categories

| Category   | Scope                                    | Approval Required     | Testing Required           |
|------------|------------------------------------------|-----------------------|----------------------------|
| Critical   | Engine algorithm, scoring logic           | 2 reviewers + QA lead | Full parity + cross-validation |
| Standard   | Security controls, data pipeline          | 1 reviewer            | Affected test suites        |
| Minor      | UI, localization, documentation           | 1 reviewer            | Visual QA                   |
| Config     | Engine thresholds, reference ranges       | 1 reviewer + clinical | Parity tests                |
| Emergency  | Security vulnerability, data loss         | Post-hoc review OK    | Minimal viable tests        |

### 5.2 Engine Directory Change Rules

All files under `Engine/` in `aura-ios` require:

1. **PR review** by at least one developer familiar with the TypeScript engine.
2. **Parity test results** included in PR description.
3. **Cross-reference** to corresponding TypeScript PR (if applicable).
4. **Version bump** if behavior changes (not just refactoring).

### 5.3 Security Directory Change Rules

All files under `AuraMedical/Security/` require:

1. **PR review** by at least one security-aware developer.
2. **No use of `print()`** -- only `os.Logger` with privacy annotations.
3. **Security test suite** run post-merge.

## 6. Build Process

### 6.1 iOS Build

| Step | Command                                    | Description                            |
|------|--------------------------------------------|----------------------------------------|
| 1    | `xcodegen generate`                        | Generate Xcode project from project.yml |
| 2    | Xcode Build (Cmd+B)                        | Compile AuraMedical scheme              |
| 3    | `xcodebuild test ...`                      | Run test suite on device                |
| 4    | `xcodebuild archive ...`                   | Create release archive                  |
| 5    | `xcodebuild -exportArchive ...`            | Export IPA for distribution             |

**Build Configuration:**
- Scheme: `AuraMedical`
- Bundle ID: `med.aura.medical`
- Team ID: `RZ9WFC8ZMG` (Auramedical Tecnologia)
- Secrets: `Debug.xcconfig` / `Release.xcconfig` (gitignored)
- Project generator: xcodegen (`project.yml`)

### 6.2 TypeScript Engine Build

| Step | Command            | Description                    |
|------|--------------------|--------------------------------|
| 1    | `npm install`      | Install dependencies           |
| 2    | `npm run build`    | TypeScript compilation         |
| 3    | `npm test`         | Run vitest parity + unit tests |
| 4    | `npm publish`      | Publish to npm (if standalone) |

### 6.3 Backend Build

| Step | Command            | Description                    |
|------|--------------------|--------------------------------|
| 1    | `npm install`      | Install dependencies           |
| 2    | `npm run build`    | TypeScript compilation         |
| 3    | `npm test`         | Run vitest tests               |
| 4    | `npm start`        | Start Hono server              |

## 7. Release Process

### 7.1 Release Pipeline

```
1. Feature freeze
   |
2. Version bump (app + engine if changed)
   |
3. Full test suite pass (parity + security + integration)
   |
4. Code coverage verification (Engine >80%, Overall >60%)
   |
5. Clinical advisor sign-off (if engine changes)
   |
6. Create git tag: v{X.Y.Z}
   |
7. Build release archive
   |
8. TestFlight internal distribution
   |
9. QA regression testing (3-5 business days)
   |
10. App Store submission
    |
11. Post-release monitoring (cross-validation logs, crash reports)
```

### 7.2 Release Artifacts

| Artifact                   | Storage          | Retention  |
|----------------------------|------------------|------------|
| Git tag                    | GitHub           | Permanent  |
| Xcode archive (.xcarchive) | Local + backup   | 2 years    |
| IPA file                   | App Store Connect | Per Apple  |
| Test results (.xcresult)   | CI artifacts     | 1 year     |
| Build log                  | CI artifacts     | 1 year     |
| CHANGELOG.md               | Git repository   | Permanent  |

### 7.3 Hotfix Process

For critical issues requiring immediate release:

1. Branch from latest release tag.
2. Apply minimal fix.
3. Run affected test suites (minimum: parity tests + security tests).
4. Emergency PR review (can be post-hoc for P0 security issues).
5. Tag as `v{X.Y.Z+1}`.
6. Expedited App Store review request.

## 8. Tools

| Tool        | Purpose                              | Version          |
|-------------|--------------------------------------|------------------|
| Xcode       | iOS IDE, build, test, archive         | 16.x             |
| xcodegen    | Xcode project generation from YAML   | Latest stable     |
| Git         | Version control                       | 2.x              |
| GitHub      | Repository hosting, PR reviews         | Cloud             |
| npm         | TypeScript dependency management       | 10.x             |
| Vitest      | TypeScript testing framework           | Latest stable     |
| XCTest      | Swift testing framework                | Built-in Xcode    |
| Supabase    | Backend-as-a-Service, database         | Cloud             |
| Render      | Backend hosting                        | Starter ($7/mo)   |
| App Store Connect | iOS app distribution             | Cloud             |

## 9. Audit Trail

All configuration changes are traceable through:

1. **Git history:** Every file change is committed with author, timestamp, and message.
2. **PR reviews:** All changes go through pull request review with approval records.
3. **Engine audit log:** Every computation logged to `ai_audit_log` with engine version.
4. **Cross-validation log:** Backend logs every cross-validation request with delta.
5. **Release tags:** Each release is immutably tagged in git.

---

**Approval Signatures:**

| Role                    | Name | Date | Signature |
|-------------------------|------|------|-----------|
| Quality Manager         |      |      |           |
| Software Development Lead |    |      |           |
| Configuration Manager   |      |      |           |
