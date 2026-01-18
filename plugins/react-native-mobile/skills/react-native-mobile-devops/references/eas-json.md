# eas.json Configuration Reference

## File Location

Place `eas.json` in the project root (same level as `package.json`).

## Complete Structure

```json
{
  "cli": {
    "version": ">= 16.3.0",
    "appVersionSource": "remote",
    "promptToConfigurePushNotifications": false
  },
  "build": {
    // Build profiles
  },
  "submit": {
    // Submit profiles
  }
}
```

## CLI Configuration

| Property | Type | Description |
|----------|------|-------------|
| `version` | string | Required EAS CLI version |
| `appVersionSource` | `"local"` \| `"remote"` | Version number source (remote enables autoIncrement) |
| `promptToConfigurePushNotifications` | boolean | Prompt for push notification setup |

## Build Profiles

### Profile Structure

```json
{
  "build": {
    "profile_name": {
      // Common options
      "extends": "base_profile",
      "distribution": "store" | "internal" | "simulator",
      "developmentClient": true | false,
      "channel": "channel_name",
      "autoIncrement": true | false | "version" | "buildNumber",

      // Environment
      "env": {
        "VAR_NAME": "value"
      },

      // Build settings
      "node": "18.18.0",
      "yarn": "1.22.19",
      "pnpm": "8.9.0",
      "bun": "1.0.0",
      "cache": {
        "key": "custom-cache-key",
        "paths": ["./custom-cache-dir"]
      },

      // Platform-specific
      "android": { },
      "ios": { }
    }
  }
}
```

### Distribution Types

| Value | Description | Use Case |
|-------|-------------|----------|
| `store` | App Store / Play Store | Production releases |
| `internal` | Ad-hoc / Internal testing | Team testing, QA |
| `simulator` | iOS Simulator only | Development |

### Common Profiles

```json
{
  "build": {
    "base": {
      "node": "18.18.0",
      "env": {
        "EXPO_PUBLIC_API_URL": "https://api.example.com"
      }
    },

    "development": {
      "extends": "base",
      "developmentClient": true,
      "distribution": "internal",
      "env": {
        "APP_ENV": "development"
      },
      "ios": {
        "simulator": true
      },
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleDebug"
      }
    },

    "preview": {
      "extends": "base",
      "distribution": "internal",
      "channel": "preview",
      "env": {
        "APP_ENV": "preview"
      },
      "android": {
        "buildType": "apk"
      }
    },

    "staging": {
      "extends": "base",
      "distribution": "internal",
      "channel": "staging",
      "env": {
        "APP_ENV": "staging",
        "EXPO_PUBLIC_API_URL": "https://api.staging.example.com"
      }
    },

    "production": {
      "extends": "base",
      "channel": "production",
      "autoIncrement": true,
      "env": {
        "APP_ENV": "production"
      }
    }
  }
}
```

### Android-Specific Options

```json
{
  "android": {
    "buildType": "apk" | "app-bundle",
    "gradleCommand": ":app:bundleRelease",
    "image": "default" | "latest" | "ubuntu-22.04-jdk-17-ndk-r26b",
    "ndk": "26.1.10909125",
    "resourceClass": "medium" | "large" | "m-medium" | "m-large",

    // Credentials
    "withoutCredentials": true,
    "credentialsSource": "local" | "remote",

    // Version
    "applicationArchivePath": "android/app/build/outputs/bundle/release/*.aab"
  }
}
```

### iOS-Specific Options

```json
{
  "ios": {
    "simulator": true | false,
    "enterpriseProvisioning": "universal" | "adhoc",
    "image": "default" | "latest" | "macos-sonoma-14.5-xcode-15.4",
    "resourceClass": "medium" | "large" | "m-medium" | "m-large" | "intel-medium",

    // Credentials
    "credentialsSource": "local" | "remote",
    "autoIncrement": true | "buildNumber",

    // Build settings
    "scheme": "MyApp",
    "buildConfiguration": "Release",
    "applicationArchivePath": "ios/build/*.ipa"
  }
}
```

## Submit Profiles

### Structure

```json
{
  "submit": {
    "profile_name": {
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "production" | "beta" | "alpha" | "internal",
        "releaseStatus": "completed" | "draft" | "halted" | "inProgress",
        "changesNotSentForReview": false
      },
      "ios": {
        "ascApiKeyPath": "./asc-api-key.p8",
        "ascApiKeyIssuerId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "ascApiKeyId": "XXXXXXXXXX",
        "appleTeamId": "XXXXXXXXXX"
      }
    }
  }
}
```

### Production Submit Profile

```json
{
  "submit": {
    "production": {
      "android": {
        "track": "production",
        "releaseStatus": "completed"
      },
      "ios": {
        "ascApiKeyPath": "./credentials/asc-api-key.p8",
        "ascApiKeyIssuerId": "your-issuer-id",
        "ascApiKeyId": "your-key-id"
      }
    },

    "beta": {
      "android": {
        "track": "beta",
        "releaseStatus": "completed"
      },
      "ios": {
        "ascApiKeyPath": "./credentials/asc-api-key.p8",
        "ascApiKeyIssuerId": "your-issuer-id",
        "ascApiKeyId": "your-key-id"
      }
    }
  }
}
```

## Environment Variables

### Using Secrets

Reference EAS Secrets with `@` prefix:

```json
{
  "build": {
    "production": {
      "env": {
        "SENTRY_DSN": "@SENTRY_DSN",
        "API_SECRET": "@API_SECRET"
      }
    }
  }
}
```

### Create Secrets

```bash
eas secret:create --name SENTRY_DSN --value "https://..." --scope project
eas secret:create --name API_SECRET --value "secret123" --scope project
```

## Complete Example

```json
{
  "cli": {
    "version": ">= 16.3.0",
    "appVersionSource": "remote"
  },
  "build": {
    "base": {
      "node": "18.18.0",
      "cache": {
        "key": "node-modules-{{ checksum \"package-lock.json\" }}"
      }
    },
    "development": {
      "extends": "base",
      "developmentClient": true,
      "distribution": "internal",
      "channel": "development",
      "env": {
        "APP_ENV": "development",
        "EXPO_PUBLIC_API_URL": "http://localhost:3000"
      },
      "ios": {
        "simulator": true
      },
      "android": {
        "buildType": "apk",
        "withoutCredentials": true
      }
    },
    "preview": {
      "extends": "base",
      "distribution": "internal",
      "channel": "preview",
      "env": {
        "APP_ENV": "preview",
        "EXPO_PUBLIC_API_URL": "https://api.preview.example.com"
      },
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "extends": "base",
      "channel": "production",
      "autoIncrement": true,
      "env": {
        "APP_ENV": "production",
        "EXPO_PUBLIC_API_URL": "https://api.example.com",
        "SENTRY_DSN": "@SENTRY_DSN"
      },
      "android": {
        "buildType": "app-bundle"
      },
      "ios": {
        "autoIncrement": "buildNumber"
      }
    }
  },
  "submit": {
    "production": {
      "android": {
        "track": "production",
        "releaseStatus": "completed"
      },
      "ios": {
        "ascApiKeyPath": "./credentials/asc-api-key.p8",
        "ascApiKeyIssuerId": "your-issuer-id",
        "ascApiKeyId": "your-key-id"
      }
    }
  }
}
```
