# Reusable Workflows

Three shared CI/CD workflows for all OpenClaw iOS repos, hosted at `Diomandeee/.github`.

## Quick Start — New Repo

1. Add required secrets to your repo (Settings > Secrets > Actions). See [Secrets](#secrets) below.
2. Create `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
jobs:
  ci:
    uses: Diomandeee/.github/.github/workflows/reusable-ci-swift.yml@main
    with:
      scheme: 'YourAppName'
```

3. (Optional) Create `.github/workflows/testflight.yml`:

```yaml
name: TestFlight
on:
  push:
    tags: ['v*']
jobs:
  upload:
    uses: Diomandeee/.github/.github/workflows/reusable-testflight.yml@main
    with:
      scheme: 'YourAppName'
      bundle-id: 'com.openclaw.yourapp'
    secrets: inherit
```

## Workflows

| File | Purpose | Runner | Required Secrets |
|------|---------|--------|-----------------|
| `reusable-ci-swift.yml` | Build + test + SwiftLint | macOS + ubuntu | None |
| `reusable-testflight.yml` | Archive + TestFlight upload | macOS | See below |
| `reusable-version-bump.yml` | SemVer bump + git tag | ubuntu | None |

## Inputs

### CI (`reusable-ci-swift.yml`)

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `xcode-version` | string | `16.2` | Xcode version for build/test |
| `scheme` | string | (auto) | Xcode scheme; auto-detected if empty |
| `test-destination` | string | `platform=iOS Simulator,name=iPhone 16` | Simulator destination |
| `project-path` | string | `.` | Subdirectory with .xcodeproj or Package.swift |
| `run-tests` | boolean | `true` | Set false for build-only |
| `runner` | string | `macos-14` | Runner image for build job |

### TestFlight (`reusable-testflight.yml`)

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `scheme` | string | (required) | Xcode scheme to archive |
| `bundle-id` | string | (required) | App bundle identifier |
| `xcode-version` | string | `16.2` | Xcode version |
| `project-path` | string | `.` | Project subdirectory |
| `build-number-override` | string | (auto) | Manual build number (positive integer) |
| `export-options-path` | string | `ExportOptions.plist` | Path to export plist |
| `project-flag` | string | | `-workspace`/`-project` flag |
| `supabase-log` | boolean | `false` | Log delivery to Supabase |
| `runner` | string | `macos-14` | Runner image |
| `team-id` | string | `8643C988C4` | Apple Developer Team ID |

### Version Bump (`reusable-version-bump.yml`)

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `bump-type` | string | `patch` | `major`, `minor`, or `patch` |
| `tag-prefix` | string | `v` | Git tag prefix |
| `project-path` | string | `.` | Subdirectory with project.yml |

## Secrets

### Required for TestFlight

| Secret | How to Get It |
|--------|--------------|
| `BUILD_CERTIFICATE_BASE64` | Export .p12 from Keychain Access, then `base64 -i cert.p12` |
| `P12_PASSWORD` | Password used when exporting the .p12 |
| `KEYCHAIN_PASSWORD` | Any strong random string (used only during CI) |
| `ASC_KEY_BASE64` | Download .p8 from ASC > Users & Access > Keys, then `base64 -i AuthKey_XXX.p8` |
| `ASC_KEY_ID` | Key ID shown in ASC > Users & Access > Keys |
| `ASC_ISSUER_ID` | Issuer ID shown on the same ASC page |

### Optional

| Secret | Purpose |
|--------|---------|
| `SUPABASE_URL` | Supabase project URL for delivery logging |
| `SUPABASE_KEY` | Supabase anon/service key |
| `GH_PAT` | PAT with `repo` scope to trigger downstream workflows on version-bump push |

## Security

- All actions are pinned to commit SHAs (not mutable tags)
- All workflow inputs are validated before use (path traversal, format checks)
- All secrets are passed via `env:` variables, never interpolated into `run:` blocks
- Concurrency groups prevent duplicate uploads and tag races
- Keychain and API key paths are run-unique to prevent collision on self-hosted runners
- Dependabot is configured to auto-update action SHAs weekly
