# Fastlane Integration

Fastlane can complement EAS for tasks like screenshots, metadata management, and advanced certificate handling.

## Setup

### Install Fastlane

```bash
# Using Bundler (recommended)
cd ios
bundle init
echo 'gem "fastlane"' >> Gemfile
bundle install

# Or globally
brew install fastlane
```

### Initialize Fastlane

```bash
cd ios
fastlane init
```

## Directory Structure

```
ios/
├── fastlane/
│   ├── Fastfile           # Lane definitions
│   ├── Appfile            # App configuration
│   ├── Matchfile          # Match configuration (optional)
│   ├── Deliverfile        # Deliver configuration (optional)
│   └── metadata/          # App Store metadata
│       ├── en-US/
│       │   ├── description.txt
│       │   ├── keywords.txt
│       │   └── release_notes.txt
│       └── screenshots/
└── Gemfile
```

## Appfile Configuration

```ruby
# ios/fastlane/Appfile

app_identifier("com.example.myapp")
apple_id("developer@example.com")
team_id("TEAM_ID")

# For App Store Connect API
# Run: fastlane spaceship token create --team-id TEAM_ID
for_platform :ios do
  app_identifier("com.example.myapp")
end
```

## Common Lanes

### Fastfile

```ruby
# ios/fastlane/Fastfile

default_platform(:ios)

platform :ios do
  # ─────────────────────────────────────────────────────────────
  # Screenshots
  # ─────────────────────────────────────────────────────────────
  desc "Capture screenshots for App Store"
  lane :screenshots do
    capture_screenshots(
      workspace: "MyApp.xcworkspace",
      scheme: "MyApp",
      output_directory: "./fastlane/screenshots",
      clear_previous_screenshots: true,
      devices: [
        "iPhone 15 Pro Max",
        "iPhone 15 Pro",
        "iPhone SE (3rd generation)",
        "iPad Pro (12.9-inch) (6th generation)"
      ],
      languages: ["en-US", "ja", "zh-Hans"]
    )
    frame_screenshots(white: true)
  end

  # ─────────────────────────────────────────────────────────────
  # Metadata
  # ─────────────────────────────────────────────────────────────
  desc "Upload metadata to App Store Connect"
  lane :upload_metadata do
    deliver(
      skip_binary_upload: true,
      skip_screenshots: false,
      force: true,
      metadata_path: "./fastlane/metadata",
      screenshots_path: "./fastlane/screenshots"
    )
  end

  desc "Download metadata from App Store Connect"
  lane :download_metadata do
    deliver(
      download_metadata: true,
      metadata_path: "./fastlane/metadata"
    )
  end

  # ─────────────────────────────────────────────────────────────
  # TestFlight
  # ─────────────────────────────────────────────────────────────
  desc "Upload to TestFlight"
  lane :beta do |options|
    ipa_path = options[:ipa] || "../build/MyApp.ipa"

    upload_to_testflight(
      ipa: ipa_path,
      skip_waiting_for_build_processing: true,
      distribute_external: false,
      changelog: options[:changelog] || "Bug fixes and improvements"
    )
  end

  desc "Distribute TestFlight build to external testers"
  lane :distribute_testflight do |options|
    upload_to_testflight(
      distribute_external: true,
      groups: ["External Testers"],
      changelog: options[:changelog] || "New beta release"
    )
  end

  # ─────────────────────────────────────────────────────────────
  # App Store
  # ─────────────────────────────────────────────────────────────
  desc "Submit to App Store Review"
  lane :release do |options|
    ipa_path = options[:ipa] || "../build/MyApp.ipa"

    deliver(
      ipa: ipa_path,
      submit_for_review: true,
      automatic_release: false,
      force: true,
      precheck_include_in_app_purchases: false,
      submission_information: {
        add_id_info_uses_idfa: false,
        content_rights_has_rights: true,
        content_rights_contains_third_party_content: true
      }
    )
  end

  # ─────────────────────────────────────────────────────────────
  # Certificates & Profiles (Match)
  # ─────────────────────────────────────────────────────────────
  desc "Sync certificates and profiles"
  lane :sync_certs do
    match(type: "appstore", readonly: true)
    match(type: "adhoc", readonly: true)
    match(type: "development", readonly: true)
  end

  desc "Create new certificates and profiles"
  lane :create_certs do
    match(type: "appstore", force: true)
    match(type: "adhoc", force: true)
    match(type: "development", force: true)
  end

  # ─────────────────────────────────────────────────────────────
  # Version Management
  # ─────────────────────────────────────────────────────────────
  desc "Bump version number"
  lane :bump_version do |options|
    increment_version_number(
      bump_type: options[:type] || "patch"  # major, minor, patch
    )
  end

  desc "Bump build number"
  lane :bump_build do
    increment_build_number(
      build_number: latest_testflight_build_number + 1
    )
  end
end
```

## Android Fastlane

```ruby
# android/fastlane/Fastfile

default_platform(:android)

platform :android do
  desc "Upload to Play Store Internal Track"
  lane :internal do |options|
    aab_path = options[:aab] || "../app/build/outputs/bundle/release/app-release.aab"

    upload_to_play_store(
      track: "internal",
      aab: aab_path,
      skip_upload_apk: true,
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end

  desc "Promote internal to beta"
  lane :promote_to_beta do
    upload_to_play_store(
      track: "internal",
      track_promote_to: "beta",
      skip_upload_apk: true,
      skip_upload_aab: true
    )
  end

  desc "Upload metadata and screenshots"
  lane :upload_metadata do
    upload_to_play_store(
      skip_upload_aab: true,
      skip_upload_apk: true,
      metadata_path: "./fastlane/metadata/android"
    )
  end

  desc "Capture screenshots"
  lane :screenshots do
    capture_android_screenshots(
      locales: ["en-US", "ja-JP"],
      clear_previous_screenshots: true,
      app_apk_path: "../app/build/outputs/apk/debug/app-debug.apk",
      tests_apk_path: "../app/build/outputs/apk/androidTest/debug/app-debug-androidTest.apk"
    )
  end
end
```

## Using Fastlane in EAS Workflows

### Screenshots Workflow

```yaml
# .eas/workflows/screenshots.yml
name: Capture Screenshots

on:
  workflow_dispatch:

defaults:
  tools:
    fastlane: 2.224.0

jobs:
  ios_screenshots:
    name: iOS Screenshots
    runs_on: macos-large
    steps:
      - uses: eas/checkout
      - uses: eas/install_node_modules

      - name: Install Fastlane Dependencies
        run: bundle install
        working_directory: ./ios

      - name: Capture Screenshots
        run: bundle exec fastlane screenshots
        working_directory: ./ios

      - name: Upload Screenshots Artifact
        run: |
          tar -czf screenshots.tar.gz fastlane/screenshots
          eas artifact:upload screenshots.tar.gz
        working_directory: ./ios
```

### Metadata Upload Workflow

```yaml
# .eas/workflows/metadata.yml
name: Upload Metadata

on:
  workflow_dispatch:
    inputs:
      platform:
        type: choice
        options: [ios, android, all]
        default: all

defaults:
  tools:
    fastlane: 2.224.0

jobs:
  upload_ios_metadata:
    name: Upload iOS Metadata
    if: ${{ inputs.platform == 'ios' || inputs.platform == 'all' }}
    runs_on: macos-medium
    steps:
      - uses: eas/checkout
      - name: Install Dependencies
        run: bundle install
        working_directory: ./ios
      - name: Upload Metadata
        run: bundle exec fastlane upload_metadata
        working_directory: ./ios
        env:
          FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD: ${{ secrets.APPLE_APP_PASSWORD }}

  upload_android_metadata:
    name: Upload Android Metadata
    if: ${{ inputs.platform == 'android' || inputs.platform == 'all' }}
    runs_on: linux-medium
    steps:
      - uses: eas/checkout
      - name: Install Dependencies
        run: bundle install
        working_directory: ./android
      - name: Upload Metadata
        run: bundle exec fastlane upload_metadata
        working_directory: ./android
        env:
          SUPPLY_JSON_KEY: ${{ secrets.PLAY_STORE_JSON_KEY }}
```

## Match (Certificate Management)

### Matchfile

```ruby
# ios/fastlane/Matchfile

git_url("git@github.com:company/certificates.git")
storage_mode("git")

type("appstore")  # Default type

app_identifier(["com.example.myapp"])
username("developer@example.com")
team_id("TEAM_ID")

# For multiple apps
# app_identifier([
#   "com.example.myapp",
#   "com.example.myapp.widget"
# ])
```

### Using Match

```bash
# First time setup (creates certs)
fastlane match appstore
fastlane match adhoc
fastlane match development

# Subsequent runs (read-only)
fastlane match appstore --readonly
```

## Environment Variables

```bash
# App Store Connect API Key
FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD=xxxx-xxxx-xxxx-xxxx

# Play Store
SUPPLY_JSON_KEY=/path/to/key.json
# Or as base64
SUPPLY_JSON_KEY_DATA=base64_encoded_key

# Match
MATCH_PASSWORD=encryption_password
MATCH_GIT_BASIC_AUTHORIZATION=base64(username:token)

# General
FASTLANE_USER=developer@example.com
FASTLANE_PASSWORD=your_password
```

## Tips

### Using App Store Connect API Key

```ruby
# More reliable than password auth
api_key = app_store_connect_api_key(
  key_id: "KEY_ID",
  issuer_id: "ISSUER_ID",
  key_filepath: "./AuthKey.p8"
)

deliver(api_key: api_key)
upload_to_testflight(api_key: api_key)
```

### Skip Waiting for Processing

```ruby
upload_to_testflight(
  skip_waiting_for_build_processing: true  # Don't wait for Apple
)
```

### Slack Notifications

```ruby
slack(
  message: "App successfully uploaded!",
  slack_url: ENV["SLACK_WEBHOOK_URL"],
  success: true,
  payload: {
    "Build Number" => get_build_number,
    "Version" => get_version_number
  }
)
```
