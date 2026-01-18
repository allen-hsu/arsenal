# EAS Deploy Command

Smart deployment that determines whether to build+submit or publish an OTA update based on native code changes.

## Workflow

1. **Check for Native Changes**
   - Run `eas update:fingerprint` to detect native code changes
   - Compare with existing production builds

2. **Determine Deployment Type**
   - **Native changes detected**: Full build + store submission
   - **JS-only changes**: OTA update (faster, instant)

3. **Execute Deployment**
   - Run appropriate EAS commands
   - Notify on completion

## Decision Logic

```
┌─────────────────────────────────────┐
│     Check Native Fingerprint        │
└───────────────┬─────────────────────┘
                │
    ┌───────────┴───────────┐
    │                       │
    ▼                       ▼
┌─────────┐           ┌─────────────┐
│ Changed │           │ No Changes  │
└────┬────┘           └──────┬──────┘
     │                       │
     ▼                       ▼
┌─────────────┐       ┌─────────────┐
│ eas build   │       │ eas update  │
│ eas submit  │       │ (OTA)       │
└─────────────┘       └─────────────┘
```

## Commands Used

### For OTA Update
```bash
eas update --branch production --message "description"
```

### For Full Build
```bash
eas build --platform [platform] --profile production
eas submit --platform [platform] --latest
```

## User Prompts

1. **Confirm Environment**: production, staging
2. **Confirm Platform**: iOS, Android, or Both
3. **Update Message**: Description for the update/build
4. **Submit to Store**: Yes/No (for builds)

## Example Output

### OTA Update
```
🔍 Checking for native changes...
✅ No native changes detected

📤 Publishing OTA update to production branch...
Running: eas update --branch production --message "Bug fixes"

✅ Update published successfully!
Users will receive the update within minutes.
```

### Full Build
```
🔍 Checking for native changes...
⚠️ Native changes detected - full build required

🏗️ Building production apps...
Running: eas build --platform all --profile production

📱 Build completed! Submit to stores?
Running: eas submit --platform all --latest

✅ Apps submitted to App Store and Play Store!
```
