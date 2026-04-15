---
name: "configuring-github-actions"
description: "Configures GitHub Actions CI/CD workflows for Flutter projects. Use when creating or editing `.github/workflows/` YAML files for Flutter, integrating subosito/flutter-action, automating `flutter test`, `flutter analyze`, or `dart format` on push or pull_request events, running build_runner in CI, building Android AAB or iOS IPA artifacts, managing keystore or signing secrets in GitHub Secrets, or combining GitHub Actions with Fastlane or Codemagic for the deployment stage."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---

# Flutter combined with GitHub Actions Best Practices (CI/CD)

## Goal
Implements robust CI/CD pipelines using GitHub Actions for Flutter projects. GitHub Actions is the most popular CI/CD platform for open-source and small-to-medium projects. By configuring YAML files under `.github/workflows/`, you can achieve high automation.

## Instructions

### 1. Environment Setup & Core Setup
In all Flutter Workflows, the core package to use is the third-party maintained [subosito/flutter-action](https://github.com/subosito/flutter-action).

**Best Practices:**
1. **Bind the Flutter Version**: Never use `channel: stable` or let it automatically fetch the latest version. Explicitly specify `flutter-version` or use `flutter-version-file: pubspec.yaml` to guarantee the CI environment exactly matches the local development environment.
2. **Enable Cache**: `flutter pub get` is a time-consuming step. Always enable `cache: true`.

```yaml
name: Flutter CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build_and_test:
    runs-on: ubuntu-latest # If not building iOS, Ubuntu is the fastest and cheapest
    
    steps:
      - name: 📦 Checkout repository
        uses: actions/checkout@v4

      - name: 🐦 Setup Flutter (Standard or FVM)
        uses: subosito/flutter-action@v2
        with:
          # 🌟 Standard Best Practice: Read from pubspec.yaml
          # flutter-version-file: pubspec.yaml 
          
          # 🌟 FVM Best Practice: If your team uses FVM, read directly from the FVM config!
          flutter-version-file: .fvmrc # Or .fvm/fvm_config.json for older FVM versions
          
          cache: true # 🌟 Enable pub-cache to significantly speed up subsequent builds
          
      # [Optional] If your strict project requirements dictate that all commands MUST be executed via `fvm flutter ...` instead of natively, you must activate the FVM CLI globally:
      # - name: 🛠️ Install FVM CLI
      #   run: dart pub global activate fvm
          
      - name: ⬇️ Get packages
        # If FVM CLI is installed, this becomes: `fvm flutter pub get`
        run: flutter pub get
```

### 2. Code Generation
If the project uses packages relying on `build_runner` (like Freezed, Riverpod, go_router builder), you must execute code generation before testing or compiling.

**Best Practice:**
If it is a Pull Request check, use `build_runner build --delete-conflicting-outputs` to ensure a clean generation state.

```yaml
      - name: ⚙️ Run build_runner
        run: dart run build_runner build --delete-conflicting-outputs
```

### 3. Analyze & Test
This is the core defense line of CI (Continuous Integration), and it's recommended to strictly require passing these for every PR.

```yaml
      # 🌟 Check if code formatting follows Dart guidelines (trailing commas, indentation)
      - name: 🔎 Analyze Formatting
        run: dart format --output=none --set-exit-if-changed .

      # 🌟 Run static analysis to ensure no warnings or lint errors
      - name: 🚨 Analyze Code
        run: flutter analyze --no-fatal-warnings

      # 🌟 Run Unit and Widget Tests (If there are integration tests, set up another job with emulators)
      - name: 🧪 Run Tests
        run: flutter test --coverage
```

### 4. Build Bundle (Android Example)
If setting up a CD (Continuous Deployment) pipeline, you can automatically build an AAB after tests pass.

```yaml
      - name: 🔨 Build Android App Bundle (AAB)
        run: flutter build appbundle --release
        
      # Upload the built file to GitHub Actions Artifacts for manual download
      - name: 📤 Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: release-aab
          path: build/app/outputs/bundle/release/app-release.aab
```

### 5. Deployment and Key Management (Keystore & App Store)
The downside of GitHub Actions is that it **isn't great at handling native certificates**.
*   **Android**: You need to convert the `keystore` into a Base64 string, store it in GitHub Secrets, and use a script in the Workflow to convert it back to a file.
*   **iOS**: More complicated, requiring the handling of Certificates and Provisioning Profiles.

## Constraints
*   **GitHub Actions should primarily be used for CI**: In the Flutter ecosystem, GitHub Actions is typically only used for CI (Analyze, Test, Build_runner) to ensure code quality.
*   **Delegate specific deployments to specialized tools**: For actual building and deployment (pushing to App Store/Play Console), delegate to tools optimized for mobile development, like **Fastlane** or **Codemagic** (which natively supports Flutter).
