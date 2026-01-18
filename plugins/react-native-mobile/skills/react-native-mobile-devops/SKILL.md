---
name: react-native-mobile-devops
description: Mobile DevOps for React Native and Expo apps using EAS (Expo Application Services) and Fastlane. Use when (1) user needs to set up CI/CD for mobile apps, (2) user asks about EAS Build, Submit, Update, or Workflows, (3) user wants to automate app deployment to App Store or Play Store, (4) user mentions "EAS", "Fastlane", "mobile CI/CD", "app deployment", or "OTA updates".
---

# Mobile DevOps (EAS + Fastlane)

Expert guidance for automating mobile app builds, submissions, and over-the-air updates using EAS (Expo Application Services) and Fastlane.

## Overview

| Service           | Purpose                                                     |
| ----------------- | ----------------------------------------------------------- |
| **EAS Build**     | Cloud builds for iOS and Android                            |
| **EAS Submit**    | Automated submission to App Store / Play Store              |
| **EAS Update**    | Over-the-air (OTA) JavaScript updates                       |
| **EAS Workflows** | CI/CD pipelines with YAML configuration                     |
| **Fastlane**      | Additional automation (screenshots, metadata, certificates) |

## Quick Start

### 1. Initialize EAS

```bash
# Install EAS CLI
npm install -g eas-cli

# Login to Expo account
eas login

# Initialize EAS in your project
eas init

# Configure build profiles
eas build:configure
```

### 2. Configure eas.json

See [references/eas-json.md](references/eas-json.md) for complete configuration.

```json
{
  "cli": {
    "version": ">= 16.3.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview"
    },
    "production": {
      "channel": "production",
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {}
  }
}
```

### 3. Create Your First Workflow

Create `.eas/workflows/build.yml`:

```yaml
name: Build Apps

on:
  push:
    branches: [main]

jobs:
  build_android:
    name: Build Android
    type: build
    params:
      platform: android
      profile: production

  build_ios:
    name: Build iOS
    type: build
    params:
      platform: ios
      profile: production
```

Run manually:

```bash
eas workflow:run .eas/workflows/build.yml
```

## EAS Workflows

### YAML Structure

See [references/eas-workflows.md](references/eas-workflows.md) for complete syntax.

```yaml
name: Workflow Name

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, production]

jobs:
  job_id:
    name: Display Name
    type: build | submit | update | fingerprint | get-build
    needs: [other_job]
    if: ${{ condition }}
    environment: production
    params:
      platform: android | ios
      profile: production
```

### Pre-packaged Job Types

| Type          | Description                 | Key Params                      |
| ------------- | --------------------------- | ------------------------------- |
| `build`       | Build app binary            | `platform`, `profile`           |
| `submit`      | Submit to app stores        | `build_id`, `profile`           |
| `update`      | Publish OTA update          | `branch`, `channel`, `platform` |
| `fingerprint` | Generate native fingerprint | `environment`                   |
| `get-build`   | Find existing build         | `fingerprint_hash`, `profile`   |
| `maestro`     | Run E2E tests               | `build_id`, `flow_path`         |
| `slack`       | Send Slack notification     | `webhook_url`, `message`        |

### Production Deployment Workflow

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  # Check if native code changed
  fingerprint:
    name: Fingerprint
    type: fingerprint
    environment: production

  # Look for existing compatible build
  get_android_build:
    name: Check Android Build
    needs: [fingerprint]
    type: get-build
    params:
      fingerprint_hash: ${{ needs.fingerprint.outputs.android_fingerprint_hash }}
      profile: production

  get_ios_build:
    name: Check iOS Build
    needs: [fingerprint]
    type: get-build
    params:
      fingerprint_hash: ${{ needs.fingerprint.outputs.ios_fingerprint_hash }}
      profile: production

  # Build only if no compatible build exists
  build_android:
    name: Build Android
    needs: [get_android_build]
    if: ${{ !needs.get_android_build.outputs.build_id }}
    type: build
    params:
      platform: android
      profile: production

  build_ios:
    name: Build iOS
    needs: [get_ios_build]
    if: ${{ !needs.get_ios_build.outputs.build_id }}
    type: build
    params:
      platform: ios
      profile: production

  # Submit new builds to stores
  submit_android:
    name: Submit to Play Store
    needs: [build_android]
    type: submit
    params:
      build_id: ${{ needs.build_android.outputs.build_id }}

  submit_ios:
    name: Submit to App Store
    needs: [build_ios]
    type: submit
    params:
      build_id: ${{ needs.build_ios.outputs.build_id }}

  # Publish OTA update if build exists
  update_android:
    name: OTA Update Android
    needs: [get_android_build]
    if: ${{ needs.get_android_build.outputs.build_id }}
    type: update
    params:
      branch: production
      platform: android

  update_ios:
    name: OTA Update iOS
    needs: [get_ios_build]
    if: ${{ needs.get_ios_build.outputs.build_id }}
    type: update
    params:
      branch: production
      platform: ios
```

## EAS Commands Reference

### Build Commands

```bash
# Build for specific platform
eas build --platform ios --profile production
eas build --platform android --profile preview
eas build --platform all --profile development

# Build with auto-submit
eas build --platform ios --auto-submit

# Local build (experimental)
eas build --platform android --local

# List builds
eas build:list

# View build details
eas build:view [BUILD_ID]

# Cancel build
eas build:cancel [BUILD_ID]
```

### Submit Commands

```bash
# Submit latest build
eas submit --platform ios --latest
eas submit --platform android --latest

# Submit specific build
eas submit --platform ios --id [BUILD_ID]

# Submit local file
eas submit --platform android --path ./app.aab
```

### Update Commands

```bash
# Publish update
eas update --branch production --message "Bug fixes"

# Publish to specific channel
eas update --channel preview --message "New feature"

# List updates
eas update:list --branch production

# Rollback (republish previous update)
eas update:republish --group [UPDATE_GROUP_ID]
```

### Workflow Commands

```bash
# Run workflow
eas workflow:run .eas/workflows/deploy.yml

# Validate workflow
eas workflow:validate .eas/workflows/deploy.yml

# List workflow runs
eas workflow:runs

# View workflow logs
eas workflow:logs [WORKFLOW_ID]

# Cancel workflow
eas workflow:cancel [WORKFLOW_ID]
```

## Channels & Branches Strategy

### Recommended Setup

```
Channel          Branch          Environment
─────────────────────────────────────────────
development  →   development  →   Dev builds
preview      →   preview      →   Internal testing
staging      →   staging      →   QA / Beta
production   →   production   →   App Store / Play Store
```

### Configuration in eas.json

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "channel": "development"
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview"
    },
    "staging": {
      "distribution": "internal",
      "channel": "staging"
    },
    "production": {
      "channel": "production",
      "autoIncrement": true
    }
  }
}
```

## Credentials Management

### iOS Credentials

```bash
# Interactive credentials setup
eas credentials --platform ios

# Credentials include:
# - Distribution Certificate
# - Provisioning Profile
# - Push Notification Key
# - App Store Connect API Key (for submit)
```

### Android Credentials

```bash
# Interactive credentials setup
eas credentials --platform android

# Credentials include:
# - Keystore (upload key or legacy app signing key)
# - Google Service Account (for submit)
```

## Environment Variables

### Build-time Secrets

```bash
# Set secret
eas secret:create --name API_KEY --value "your-secret-key" --scope project

# List secrets
eas secret:list

# Delete secret
eas secret:delete API_KEY
```

### Using in eas.json

```json
{
  "build": {
    "production": {
      "env": {
        "API_URL": "https://api.production.com",
        "SENTRY_DSN": "@SENTRY_DSN"
      }
    }
  }
}
```

## Fastlane Integration

See [references/fastlane.md](references/fastlane.md) for Fastlane patterns.

### Using Fastlane in EAS Workflows

```yaml
defaults:
  tools:
    fastlane: 2.224.0

jobs:
  screenshots:
    runs_on: macos-large
    steps:
      - uses: eas/checkout
      - name: Capture Screenshots
        run: fastlane screenshots
        working_directory: ./ios
```

## Resources

- **EAS JSON Config**: [references/eas-json.md](references/eas-json.md)
- **EAS Workflows Syntax**: [references/eas-workflows.md](references/eas-workflows.md)
- **Fastlane Patterns**: [references/fastlane.md](references/fastlane.md)
- **Deployment Patterns**: [references/deployment-patterns.md](references/deployment-patterns.md)
