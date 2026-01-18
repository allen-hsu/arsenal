# EAS Workflow Command

Create or run EAS workflow files for CI/CD automation.

## Subcommands

### Create Workflow
Generate a new workflow YAML file based on common patterns.

### Run Workflow
Execute an existing workflow file.

### Validate Workflow
Check workflow syntax before running.

## Workflow Templates

### 1. Build on Push
```yaml
name: Build on Push

on:
  push:
    branches: [main]

jobs:
  build:
    type: build
    params:
      platform: all
      profile: production
```

### 2. PR Preview
```yaml
name: PR Preview

on:
  pull_request:
    branches: [main]

jobs:
  preview:
    type: build
    params:
      platform: all
      profile: preview
```

### 3. Smart Deploy
```yaml
name: Smart Deploy

on:
  push:
    branches: [main]

jobs:
  fingerprint:
    type: fingerprint
    environment: production

  check_build:
    type: get-build
    needs: [fingerprint]
    params:
      fingerprint_hash: ${{ needs.fingerprint.outputs.android_fingerprint_hash }}

  build:
    type: build
    needs: [check_build]
    if: ${{ !needs.check_build.outputs.build_id }}
    params:
      platform: all
      profile: production

  update:
    type: update
    needs: [check_build]
    if: ${{ needs.check_build.outputs.build_id }}
    params:
      branch: production
```

### 4. Nightly
```yaml
name: Nightly

on:
  schedule:
    - cron: '0 2 * * *'

jobs:
  build:
    type: build
    params:
      platform: all
      profile: staging
```

## User Prompts

For "Create Workflow":
1. **Workflow Type**: Build, PR Preview, Smart Deploy, Nightly, Custom
2. **Trigger**: Push, PR, Schedule, Manual
3. **Branch(es)**: main, develop, etc.
4. **Profile**: production, staging, preview
5. **Filename**: e.g., `deploy-production.yml`

For "Run Workflow":
1. **Select File**: List available `.yml` files in `.eas/workflows/`
2. **Confirm**: Show workflow name and trigger

## Commands

```bash
# Create workflow directory if needed
mkdir -p .eas/workflows

# Validate workflow
eas workflow:validate .eas/workflows/[file].yml

# Run workflow
eas workflow:run .eas/workflows/[file].yml

# List workflow runs
eas workflow:runs

# View logs
eas workflow:logs [WORKFLOW_ID]
```

## Example Flow

```
🔧 EAS Workflow Manager

What would you like to do?
1. Create new workflow
2. Run existing workflow
3. Validate workflow
4. View workflow runs

> 1

Select workflow template:
1. Build on Push (main → production)
2. PR Preview Builds
3. Smart Deploy (build or OTA)
4. Nightly Builds
5. Custom

> 3

Creating Smart Deploy workflow...
✅ Created: .eas/workflows/smart-deploy.yml

Would you like to run this workflow now? (y/n)
> y

Running: eas workflow:run .eas/workflows/smart-deploy.yml
```
