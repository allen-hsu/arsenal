# Deployment Patterns

Common deployment strategies for React Native + Expo apps using EAS.

## Pattern 1: Simple Build & Submit

Best for: Small teams, simple apps

```yaml
# .eas/workflows/release.yml
name: Release

on:
  push:
    tags: ['v*']

jobs:
  build_android:
    type: build
    params:
      platform: android
      profile: production

  build_ios:
    type: build
    params:
      platform: ios
      profile: production

  submit_android:
    type: submit
    needs: [build_android]
    params:
      build_id: ${{ needs.build_android.outputs.build_id }}

  submit_ios:
    type: submit
    needs: [build_ios]
    params:
      build_id: ${{ needs.build_ios.outputs.build_id }}
```

## Pattern 2: Smart Deploy (Build or OTA)

Best for: Frequent updates, cost optimization

Uses fingerprinting to determine if native code changed:
- **Native changes**: New build + store submission
- **JS-only changes**: OTA update (instant, free)

```yaml
# .eas/workflows/smart-deploy.yml
name: Smart Deploy

on:
  push:
    branches: [main]

jobs:
  fingerprint:
    type: fingerprint
    environment: production

  # ─── Android ───────────────────────────────────────────────
  check_android:
    type: get-build
    needs: [fingerprint]
    params:
      fingerprint_hash: ${{ needs.fingerprint.outputs.android_fingerprint_hash }}
      profile: production

  build_android:
    type: build
    needs: [check_android]
    if: ${{ !needs.check_android.outputs.build_id }}
    params:
      platform: android
      profile: production

  submit_android:
    type: submit
    needs: [build_android]
    params:
      build_id: ${{ needs.build_android.outputs.build_id }}

  update_android:
    type: update
    needs: [check_android]
    if: ${{ needs.check_android.outputs.build_id }}
    params:
      platform: android
      branch: production

  # ─── iOS ───────────────────────────────────────────────────
  check_ios:
    type: get-build
    needs: [fingerprint]
    params:
      fingerprint_hash: ${{ needs.fingerprint.outputs.ios_fingerprint_hash }}
      profile: production

  build_ios:
    type: build
    needs: [check_ios]
    if: ${{ !needs.check_ios.outputs.build_id }}
    params:
      platform: ios
      profile: production

  submit_ios:
    type: submit
    needs: [build_ios]
    params:
      build_id: ${{ needs.build_ios.outputs.build_id }}

  update_ios:
    type: update
    needs: [check_ios]
    if: ${{ needs.check_ios.outputs.build_id }}
    params:
      platform: ios
      branch: production
```

## Pattern 3: Multi-Environment Pipeline

Best for: Teams with staging environments

```
develop → preview builds → internal testing
staging → staging builds → QA testing
main    → production builds → App Store / Play Store
```

```yaml
# .eas/workflows/pipeline.yml
name: CI/CD Pipeline

on:
  push:
    branches: [develop, staging, main]

jobs:
  # ─── Develop → Preview ─────────────────────────────────────
  preview_build:
    name: Preview Build
    if: ${{ github.ref_name == 'develop' }}
    type: build
    params:
      platform: all
      profile: preview

  # ─── Staging ───────────────────────────────────────────────
  staging_build:
    name: Staging Build
    if: ${{ github.ref_name == 'staging' }}
    type: build
    params:
      platform: all
      profile: staging

  staging_update:
    name: Staging OTA
    if: ${{ github.ref_name == 'staging' }}
    type: update
    params:
      branch: staging
      platform: all

  # ─── Production ────────────────────────────────────────────
  production_fingerprint:
    name: Fingerprint
    if: ${{ github.ref_name == 'main' }}
    type: fingerprint
    environment: production

  production_check_android:
    type: get-build
    needs: [production_fingerprint]
    if: ${{ github.ref_name == 'main' }}
    params:
      fingerprint_hash: ${{ needs.production_fingerprint.outputs.android_fingerprint_hash }}
      profile: production

  production_check_ios:
    type: get-build
    needs: [production_fingerprint]
    if: ${{ github.ref_name == 'main' }}
    params:
      fingerprint_hash: ${{ needs.production_fingerprint.outputs.ios_fingerprint_hash }}
      profile: production

  production_build_android:
    type: build
    needs: [production_check_android]
    if: ${{ github.ref_name == 'main' && !needs.production_check_android.outputs.build_id }}
    params:
      platform: android
      profile: production

  production_build_ios:
    type: build
    needs: [production_check_ios]
    if: ${{ github.ref_name == 'main' && !needs.production_check_ios.outputs.build_id }}
    params:
      platform: ios
      profile: production

  production_submit_android:
    type: submit
    needs: [production_build_android]
    params:
      build_id: ${{ needs.production_build_android.outputs.build_id }}

  production_submit_ios:
    type: submit
    needs: [production_build_ios]
    params:
      build_id: ${{ needs.production_build_ios.outputs.build_id }}

  production_update_android:
    type: update
    needs: [production_check_android]
    if: ${{ github.ref_name == 'main' && needs.production_check_android.outputs.build_id }}
    params:
      platform: android
      branch: production

  production_update_ios:
    type: update
    needs: [production_check_ios]
    if: ${{ github.ref_name == 'main' && needs.production_check_ios.outputs.build_id }}
    params:
      platform: ios
      branch: production
```

## Pattern 4: PR Preview Builds

Best for: Code review with real builds

```yaml
# .eas/workflows/pr-preview.yml
name: PR Preview

on:
  pull_request:
    branches: [main, develop]
  pull_request_labeled:
    labels: [build-preview]

jobs:
  build:
    name: Build Preview
    type: build
    params:
      platform: all
      profile: preview
      message: 'PR #${{ github.event.pull_request.number }}: ${{ github.event.pull_request.title }}'

  comment:
    name: Post Build Links
    type: github-comment
    needs: [build]
    params:
      message: |
        ## 📱 Preview Builds Ready

        Install the preview builds to test this PR:

        | Platform | Build |
        |----------|-------|
        | Android | [Download APK](https://expo.dev/accounts/...) |
        | iOS | [Install via Expo](https://expo.dev/accounts/...) |

        > Builds expire after 30 days
```

## Pattern 5: Scheduled Nightly Builds

Best for: Daily QA builds, regression testing

```yaml
# .eas/workflows/nightly.yml
name: Nightly Builds

on:
  schedule:
    - cron: '0 2 * * 1-5'  # 2am UTC, Mon-Fri

jobs:
  build_android:
    type: build
    params:
      platform: android
      profile: staging
      message: 'Nightly #${{ github.run_number }}'

  build_ios:
    type: build
    params:
      platform: ios
      profile: staging
      message: 'Nightly #${{ github.run_number }}'

  run_tests:
    type: maestro
    needs: [build_android]
    params:
      build_id: ${{ needs.build_android.outputs.build_id }}
      flow_path: .maestro/

  notify:
    type: slack
    after: [build_android, build_ios, run_tests]
    params:
      webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      message: |
        🌙 Nightly Build Report

        Android: ${{ after.build_android.status }}
        iOS: ${{ after.build_ios.status }}
        E2E Tests: ${{ after.run_tests.status }}
```

## Pattern 6: Manual Release with Approval

Best for: Controlled releases, compliance requirements

```yaml
# .eas/workflows/manual-release.yml
name: Manual Release

on:
  workflow_dispatch:
    inputs:
      platform:
        type: choice
        options: [android, ios, all]
        default: all
      environment:
        type: choice
        options: [staging, production]
        default: staging
      submit:
        type: boolean
        description: Submit to store
        default: false

jobs:
  build_android:
    name: Build Android
    if: ${{ inputs.platform == 'android' || inputs.platform == 'all' }}
    type: build
    environment: ${{ inputs.environment }}
    params:
      platform: android
      profile: ${{ inputs.environment }}

  build_ios:
    name: Build iOS
    if: ${{ inputs.platform == 'ios' || inputs.platform == 'all' }}
    type: build
    environment: ${{ inputs.environment }}
    params:
      platform: ios
      profile: ${{ inputs.environment }}

  submit_android:
    name: Submit to Play Store
    type: submit
    needs: [build_android]
    if: ${{ inputs.submit && (inputs.platform == 'android' || inputs.platform == 'all') }}
    params:
      build_id: ${{ needs.build_android.outputs.build_id }}

  submit_ios:
    name: Submit to App Store
    type: submit
    needs: [build_ios]
    if: ${{ inputs.submit && (inputs.platform == 'ios' || inputs.platform == 'all') }}
    params:
      build_id: ${{ needs.build_ios.outputs.build_id }}
```

## Pattern 7: Hotfix Deployment

Best for: Emergency fixes

```yaml
# .eas/workflows/hotfix.yml
name: Hotfix

on:
  push:
    branches: ['hotfix/**']

jobs:
  # Quick OTA update for JS fixes
  ota_update:
    name: Emergency OTA
    type: update
    params:
      branch: production
      platform: all
      message: 'HOTFIX: ${{ github.event.head_commit.message }}'

  notify:
    type: slack
    needs: [ota_update]
    params:
      webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      message: |
        🚨 HOTFIX DEPLOYED

        Branch: ${{ github.ref_name }}
        Message: ${{ github.event.head_commit.message }}

        Update will be available to users within minutes.
```

## Channels & Branches Mapping

### Recommended Setup

```
┌─────────────┬─────────────┬─────────────────────────────────┐
│ Git Branch  │ EAS Channel │ Purpose                         │
├─────────────┼─────────────┼─────────────────────────────────┤
│ develop     │ development │ Development builds              │
│ feature/*   │ preview     │ PR preview builds               │
│ staging     │ staging     │ QA / Beta testing               │
│ main        │ production  │ Production App Store releases   │
│ hotfix/*    │ production  │ Emergency OTA updates           │
└─────────────┴─────────────┴─────────────────────────────────┘
```

### eas.json Configuration

```json
{
  "build": {
    "development": {
      "channel": "development",
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "channel": "preview",
      "distribution": "internal"
    },
    "staging": {
      "channel": "staging",
      "distribution": "internal"
    },
    "production": {
      "channel": "production",
      "autoIncrement": true
    }
  }
}
```

## Version Strategy

### Automatic Version Increment

```json
{
  "build": {
    "production": {
      "autoIncrement": true
    }
  }
}
```

### Manual Version Control

```bash
# Set version via CLI
eas build:version:set --platform ios
eas build:version:set --platform android

# Sync from EAS to local
eas build:version:sync --platform all
```

### Semantic Versioning Workflow

```yaml
# Triggered by tags like v1.2.3
on:
  push:
    tags: ['v[0-9]+.[0-9]+.[0-9]+']

jobs:
  release:
    runs_on: linux-medium
    steps:
      - uses: eas/checkout
      - name: Extract Version
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
      - name: Update app.json
        run: |
          jq '.expo.version = "${{ steps.version.outputs.version }}"' app.json > tmp.json
          mv tmp.json app.json

  build:
    type: build
    needs: [release]
    params:
      platform: all
      profile: production
```
