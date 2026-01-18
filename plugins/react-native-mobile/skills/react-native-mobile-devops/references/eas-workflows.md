# EAS Workflows YAML Reference

## File Location

Place workflow files in `.eas/workflows/` directory:

```
project/
├── .eas/
│   └── workflows/
│       ├── build.yml
│       ├── deploy-production.yml
│       └── preview.yml
├── eas.json
└── package.json
```

## Top-Level Structure

```yaml
name: Workflow Name          # Required: Display name
on: { }                      # Triggers
defaults: { }                # Shared configuration
concurrency: { }             # Concurrency control
jobs: { }                    # Required: Job definitions
```

## Triggers (`on`)

### Push Trigger

```yaml
on:
  push:
    branches:
      - main
      - develop
      - 'release/**'        # Glob pattern
      - '!release/beta'     # Exclude pattern
    tags:
      - 'v*'
      - 'v[0-9]+.[0-9]+.[0-9]+'
    paths:
      - 'src/**'
      - 'app/**'
      - '!**/*.md'          # Exclude markdown files
```

### Pull Request Trigger

```yaml
on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - reopened
      - synchronize
      - labeled
    paths:
      - 'src/**'
```

### Pull Request Label Trigger

```yaml
on:
  pull_request_labeled:
    labels:
      - 'build:preview'
      - 'deploy:staging'
```

### Schedule Trigger (Cron)

```yaml
on:
  schedule:
    - cron: '0 0 * * *'     # Daily at midnight UTC
    - cron: '0 9 * * 1'     # Every Monday at 9am UTC
```

### Manual Trigger with Inputs

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        description: 'Deployment environment'
        required: true
        default: staging
        options:
          - staging
          - production
      skip_tests:
        type: boolean
        description: 'Skip E2E tests'
        default: false
      message:
        type: string
        description: 'Update message'
        required: false
```

## Jobs

### Pre-packaged Job Types

#### Build Job

```yaml
jobs:
  build:
    name: Build App
    type: build
    environment: production    # Optional: linked environment
    params:
      platform: android | ios  # Required
      profile: production      # Optional, default: production
      message: 'CI Build'      # Optional
```

**Outputs:**
- `build_id` - Build identifier
- `app_version` - App version string
- `app_build_version` - Build number
- `fingerprint_hash` - Native fingerprint

#### Submit Job

```yaml
jobs:
  submit:
    name: Submit to Store
    type: submit
    needs: [build]
    params:
      build_id: ${{ needs.build.outputs.build_id }}  # Required
      profile: production    # Optional
```

**Outputs:**
- `apple_app_id`
- `ios_bundle_identifier`
- `android_package_id`

#### Update Job (OTA)

```yaml
jobs:
  update:
    name: Publish OTA Update
    type: update
    params:
      message: 'Bug fixes'           # Optional
      platform: all | android | ios  # Optional, default: all
      branch: production             # Optional
      channel: production            # Optional (cannot use with branch)
      private_key_path: ./key.pem    # Optional
      upload_sentry_sourcemaps: true # Optional
```

**Outputs:**
- `first_update_group_id`
- `updates_json` - JSON array of update info

#### Fingerprint Job

```yaml
jobs:
  fingerprint:
    name: Generate Fingerprint
    type: fingerprint
    environment: production

# Outputs:
# - android_fingerprint_hash
# - ios_fingerprint_hash
# - android_fingerprint_sources
# - ios_fingerprint_sources
```

#### Get Build Job

```yaml
jobs:
  get_build:
    name: Find Existing Build
    type: get-build
    needs: [fingerprint]
    params:
      fingerprint_hash: ${{ needs.fingerprint.outputs.android_fingerprint_hash }}
      profile: production

# Outputs:
# - build_id (empty if no matching build)
```

#### Maestro Test Job

```yaml
jobs:
  e2e_tests:
    name: Run E2E Tests
    type: maestro
    needs: [build]
    params:
      build_id: ${{ needs.build.outputs.build_id }}
      flow_path: .maestro/           # Directory with test flows
      # Or specify individual flows:
      # flow_path: |
      #   .maestro/login.yaml
      #   .maestro/checkout.yaml
```

#### Slack Notification Job

```yaml
jobs:
  notify:
    name: Send Slack Notification
    type: slack
    params:
      webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      message: 'Build completed!'
```

#### GitHub Comment Job

```yaml
jobs:
  comment:
    name: Post PR Comment
    type: github-comment
    needs: [build_android, build_ios]
    params:
      message: 'Builds ready for testing'
      build_ids:
        - ${{ needs.build_android.outputs.build_id }}
        - ${{ needs.build_ios.outputs.build_id }}
```

### Custom Jobs

```yaml
jobs:
  custom_job:
    name: Custom Task
    runs_on: linux-medium | linux-large | macos-medium | macos-large
    environment: production
    env:
      NODE_ENV: production
    steps:
      - uses: eas/checkout
      - uses: eas/install_node_modules
      - name: Run Tests
        run: npm test
      - name: Build
        run: npm run build
        working_directory: ./packages/app
```

### Built-in Steps (`uses`)

| Step | Description |
|------|-------------|
| `eas/checkout` | Clone repository |
| `eas/install_node_modules` | Install dependencies |
| `eas/prebuild` | Run expo prebuild |
| `eas/download_build` | Download build artifact |
| `eas/use_npm_token` | Configure private npm |
| `eas/send_slack_message` | Post to Slack |
| `eas/download_artifact` | Get job artifact |

## Job Dependencies

### Sequential Execution (`needs`)

```yaml
jobs:
  build:
    type: build
    params:
      platform: ios

  submit:
    type: submit
    needs: [build]  # Waits for build to succeed
    params:
      build_id: ${{ needs.build.outputs.build_id }}
```

### Run After (regardless of status)

```yaml
jobs:
  build:
    type: build
    params:
      platform: ios

  notify:
    type: slack
    after: [build]  # Runs even if build fails
    params:
      message: 'Build finished: ${{ after.build.status }}'
```

## Conditional Execution (`if`)

```yaml
jobs:
  build:
    type: build
    if: ${{ github.ref_name == 'main' }}
    params:
      platform: ios

  submit:
    type: submit
    needs: [build]
    if: ${{ success() && github.ref_name == 'main' }}
    params:
      build_id: ${{ needs.build.outputs.build_id }}

  update:
    type: update
    needs: [get_build]
    if: ${{ needs.get_build.outputs.build_id }}  # Only if build exists
    params:
      branch: production
```

## Expressions

### Context Variables

```yaml
# GitHub context
${{ github.ref }}              # refs/heads/main
${{ github.ref_name }}         # main
${{ github.sha }}              # commit SHA
${{ github.event_name }}       # push, pull_request, etc.

# Workflow context
${{ workflow.name }}
${{ workflow.id }}

# Job outputs
${{ needs.job_id.outputs.output_name }}
${{ after.job_id.status }}     # success, failure, cancelled

# Step outputs (custom jobs)
${{ steps.step_id.outputs.value }}

# Inputs (workflow_dispatch)
${{ inputs.environment }}

# Environment variables
${{ env.VAR_NAME }}

# Secrets
${{ secrets.SECRET_NAME }}
```

### Functions

```yaml
# Status checks
if: ${{ success() }}
if: ${{ failure() }}
if: ${{ always() }}

# String functions
${{ contains(github.ref, 'release') }}
${{ startsWith(github.ref_name, 'feature/') }}
${{ endsWith(github.ref_name, '-hotfix') }}
${{ replaceAll(github.ref_name, '/', '-') }}
${{ substring(github.sha, 0, 7) }}

# JSON functions
${{ toJSON(needs.build.outputs) }}
${{ fromJSON(needs.update.outputs.updates_json) }}

# File hash
${{ hashFiles('**/package-lock.json') }}
```

## Defaults

```yaml
defaults:
  run:
    working_directory: ./apps/mobile
    shell: bash
  tools:
    node: '18.18.0'
    yarn: '1.22.19'
    pnpm: '8.9.0'
    bun: '1.0.0'
    fastlane: '2.224.0'
```

## Concurrency

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## Complete Examples

### PR Preview Build

```yaml
name: PR Preview

on:
  pull_request:
    branches: [main]

jobs:
  build_preview:
    name: Build Preview
    type: build
    params:
      platform: all
      profile: preview
      message: 'PR #${{ github.event.pull_request.number }}'

  comment:
    name: Post Build Links
    type: github-comment
    needs: [build_preview]
    params:
      message: |
        ## Preview Builds Ready

        Install the preview builds to test this PR.
```

### Nightly Builds

```yaml
name: Nightly Builds

on:
  schedule:
    - cron: '0 2 * * *'  # 2am UTC daily

jobs:
  build_android:
    name: Nightly Android
    type: build
    params:
      platform: android
      profile: staging
      message: 'Nightly build ${{ github.run_number }}'

  build_ios:
    name: Nightly iOS
    type: build
    params:
      platform: ios
      profile: staging
      message: 'Nightly build ${{ github.run_number }}'

  notify:
    name: Notify Team
    type: slack
    after: [build_android, build_ios]
    params:
      webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      message: |
        Nightly builds completed
        Android: ${{ after.build_android.status }}
        iOS: ${{ after.build_ios.status }}
```
