# Create Flutter Build

Sets up Java and Flutter, fetches dependencies, and runs Flutter doctor, analyzer, and tests. Use this action in a workflow to validate your Flutter project (dependencies, tooling, static analysis, and unit tests) before building or deploying.

## What it does

1. **Checkout** – Checks out your repo
2. **Setup Java** – Installs Temurin JDK (default: 17)
3. **Setup Flutter** – Installs Flutter (default: 3.24.0, stable channel)
4. **Get dependencies** – Runs `flutter pub get`
5. **Verify installation** (optional) – Runs `flutter doctor -v`
6. **Run analyzer** (optional) – Runs `flutter analyze` with optional `--no-fatal-infos` and `--no-fatal-warnings`
7. **Run tests** (optional) – Runs `flutter test`

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `flutter_version` | No | `3.24.0` | Flutter version to install |
| `flutter_channel` | No | `stable` | Flutter channel: `stable`, `beta`, or `master` |
| `java_version` | No | `17` | Java version for setup-java |
| `run_doctor` | No | `true` | Whether to run `flutter doctor -v` |
| `run_analyze` | No | `true` | Whether to run `flutter analyze` |
| `analyze_no_fatal_infos` | No | `true` | Pass `--no-fatal-infos` to `flutter analyze` |
| `analyze_no_fatal_warnings` | No | `true` | Pass `--no-fatal-warnings` to `flutter analyze` |
| `run_tests` | No | `true` | Whether to run `flutter test` |

## Usage

### Basic (defaults: doctor, analyze, tests)

```yaml
test:
  name: Run Tests
  runs-on: ubuntu-latest
  steps:
    - uses: ./.github/actions/create-flutter-build
```

### From this repo (another project)

```yaml
test:
  name: Run Tests
  runs-on: ubuntu-latest
  steps:
    - uses: wakeel39/github-actions/.github/actions/create-flutter-build@main
```

### With Flutter version from env

```yaml
env:
  FLUTTER_VERSION: '3.24.0'
jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - uses: ./.github/actions/create-flutter-build
        with:
          flutter_version: ${{ env.FLUTTER_VERSION }}
```

### Analyzer only (no doctor, no tests)

```yaml
- uses: ./.github/actions/create-flutter-build
  with:
    run_doctor: 'false'
    run_tests: 'false'
```

### Tests only (no doctor, no analyze)

```yaml
- uses: ./.github/actions/create-flutter-build
  with:
    run_doctor: 'false'
    run_analyze: 'false'
```

### Strict analyzer (fail on infos and warnings)

```yaml
- uses: ./.github/actions/create-flutter-build
  with:
    analyze_no_fatal_infos: 'false'
    analyze_no_fatal_warnings: 'false'
```

## Requirements

- **Runner**: Ubuntu (e.g. `ubuntu-latest`); the action uses Bash.
- Your project must have a `pubspec.yaml` and be a valid Flutter/Dart project.

## Using in your own repo

1. In your workflow, add a job (e.g. `test`) that runs on `ubuntu-latest`.
2. Use either:
   - **Same repo**: `uses: ./.github/actions/create-flutter-build` (after copying this action into your repo), or
   - **This repo**: `uses: wakeel39/github-actions/.github/actions/create-flutter-build@main`
3. Optionally pass `flutter_version`, `java_version`, or toggle `run_doctor` / `run_analyze` / `run_tests` as in the examples above.
