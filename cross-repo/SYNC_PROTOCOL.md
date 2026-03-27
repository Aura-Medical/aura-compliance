# Phenomic Engine — Sync Protocol

Three repositories share the Phenomic Engine logic. They **must** stay in sync.

| Repo | Language | Role |
|------|----------|------|
| `phenomic-engine/` | TypeScript | Canonical source of truth |
| `aura-backend/` | TypeScript | Consumes engine via vendored `dist/` |
| `aura-ios/Engine/` | Swift | Manual port for on-device computation |

## When the TS Engine Changes

Follow these steps **in order**:

### 1. Bump version
- Update `version` in `phenomic-engine/package.json`
- Update `PhenomicEngine.engineVersion` in `aura-ios/Engine/PhenomicEngine.swift` to `"{version}-swift"`

### 2. Regenerate parity vectors
```bash
cd phenomic-engine
npm test          # Ensures vectors are up to date
```

### 3. Update Swift port
- Apply the same math/threshold/weight changes to `aura-ios/Engine/PhenomicEngine.swift`
- Apply config changes to `aura-ios/Engine/EngineConfig.swift`

### 4. Run parity tests
```bash
cd aura-ios
./scripts/sync-parity-vectors.sh   # Copy vectors from TS to iOS fixtures
# Then run iOS parity tests in Xcode (AuraMedicalTests/Engine/ParityTests.swift)
```

### 5. Update backend vendor
```bash
cd phenomic-engine
npm run build
cd ../aura-backend
npm run prebuild   # Copies dist/ to vendor/engine/
npm run build      # Verify compilation
```

### 6. Commit all repos
- Commit phenomic-engine changes
- Commit aura-ios Engine/ changes + updated parity vectors
- Commit aura-backend vendor/engine/ changes

## Tolerance

The backend cross-validation allows a **delta of ≤ 2 points** between TS and Swift scores.
This accounts for floating-point differences between platforms.

## Parity Vectors

Located at `phenomic-engine/__tests__/parity-vectors.json`.
Each vector has `input` and `expected` fields.
The Swift `ParityTests.swift` loads these and verifies the Swift engine produces matching output.
