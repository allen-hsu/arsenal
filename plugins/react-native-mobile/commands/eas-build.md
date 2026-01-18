# EAS Build Command

Trigger an EAS build with the appropriate profile based on the current branch or user preference.

## Workflow

1. **Detect Context**
   - Check current git branch
   - Determine appropriate build profile

2. **Confirm with User**
   - Show detected platform and profile
   - Allow override if needed

3. **Run Build**
   - Execute `eas build` command
   - Monitor build status

## Profile Selection Logic

| Branch Pattern | Default Profile |
|---------------|-----------------|
| `main`, `master` | `production` |
| `staging` | `staging` |
| `develop`, `dev` | `preview` |
| `feature/*`, `fix/*` | `preview` |
| Other | `development` |

## Command Execution

```bash
# Detect branch and suggest profile
BRANCH=$(git branch --show-current)

# Run build
eas build --platform [ios|android|all] --profile [profile]
```

## User Options

Ask user:
1. **Platform**: iOS, Android, or Both
2. **Profile**: Use detected or override
3. **Message**: Optional build message

## Example Output

```
🔍 Detected branch: develop
📦 Suggested profile: preview

Building for: iOS + Android
Profile: preview

Running: eas build --platform all --profile preview
```
