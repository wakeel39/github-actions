# Build Android App Bundle

Builds a signed Flutter Android App Bundle (AAB) and uploads it as an artifact. Use this action after tests pass (e.g. after the Flutter test action) to produce a release AAB for Play Store or other distribution.

## What it does

1. **Checkout** – Checks out your repo
2. **Setup Java** – Temurin JDK with Gradle cache (default: 17)
3. **Setup Flutter** – Installs Flutter (default: 3.24.0, stable)
4. **Get dependencies** – `flutter pub get`
5. **Clean Gradle** – `./gradlew clean --stop` in `android/`
6. **Set version** – Version name from tag (`v1.1.0` → `1.1.0`) or `1.0.{run_number}`; version code from run number
7. **Update pubspec** – Writes `version: {name}+{code}` into `pubspec.yaml`
8. **Decode keystore** – Writes base64-decoded keystore to `android/app/keystore.jks`
9. **Create key.properties** – Writes store/key passwords and alias to `android/key.properties`
10. **Build AAB** – `flutter build appbundle --release --build-number={code}`
11. **Upload artifact** – Uploads `app-release.aab` with configurable name and retention

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `keystore_base64` | Yes | - | Base64-encoded keystore file (use a secret) |
| `keystore_password` | Yes | - | Keystore password (use a secret) |
| `key_password` | Yes | - | Key password (use a secret) |
| `key_alias` | Yes | - | Key alias (use a secret) |
| `flutter_version` | No | `3.24.0` | Flutter version |
| `java_version` | No | `17` | Java version |
| `artifact_name` | No | `app-bundle` | Name of the upload-artifact |
| `retention_days` | No | `30` | Artifact retention in days |

## Outputs

| Output | Description |
|--------|-------------|
| `version_name` | App version name (e.g. `1.1.0` or `1.0.123`) |
| `version_code` | App version code (integer) |

Use these in later jobs (e.g. deploy) by giving the action an `id` and referencing `steps.<id>.outputs.version_name` and `steps.<id>.outputs.version_code`.

## Usage

### Basic (same repo)

```yaml
build:
  name: Build Android App Bundle
  needs: test
  runs-on: ubuntu-latest
  outputs:
    version_name: ${{ steps.build.outputs.version_name }}
    version_code: ${{ steps.build.outputs.version_code }}
  steps:
    - uses: ./.github/actions/build-android-appbundle
      id: build
      with:
        keystore_base64: ${{ secrets.KEYSTORE_BASE64 }}
        keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
        key_password: ${{ secrets.KEY_PASSWORD }}
        key_alias: ${{ secrets.KEY_ALIAS }}
```

### From this repo (another project)

```yaml
build:
  name: Build Android App Bundle
  needs: test
  runs-on: ubuntu-latest
  outputs:
    version_name: ${{ steps.build.outputs.version_name }}
    version_code: ${{ steps.build.outputs.version_code }}
  steps:
    - uses: wakeel39/github-actions/.github/actions/build-android-appbundle@main
      id: build
      with:
        keystore_base64: ${{ secrets.KEYSTORE_BASE64 }}
        keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
        key_password: ${{ secrets.KEY_PASSWORD }}
        key_alias: ${{ secrets.KEY_ALIAS }}
        flutter_version: ${{ env.FLUTTER_VERSION }}
```

### With custom artifact name and retention

```yaml
- uses: ./.github/actions/build-android-appbundle
  id: build
  with:
    keystore_base64: ${{ secrets.KEYSTORE_BASE64 }}
    keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
    key_password: ${{ secrets.KEY_PASSWORD }}
    key_alias: ${{ secrets.KEY_ALIAS }}
    artifact_name: app-bundle
    retention_days: 14
```

## Prerequisites

- **Secrets**: `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_PASSWORD`, `KEY_ALIAS` (or your own secret names passed as inputs).
- **Project**: Flutter Android app with `android/` and a `pubspec.yaml` that has a `version:` line.
- **Keystore**: Encode your `.jks` once: `base64 -w0 android/app/keystore.jks` (or equivalent) and store the result in a repo/organization secret.

## Version logic

- **On tag** (e.g. `refs/tags/v1.2.0`): version name = `1.2.0`, version code = `2 + run_number`.
- **On branch**: version name = `1.0.{run_number}`, version code = `2 + run_number`.

`pubspec.yaml` is updated to `version: {version_name}+{version_code}` before the build.

## Using in your own repo

1. Add repo secrets for the keystore (base64), store password, key password, and key alias.
2. Add a job that `needs: test` (or your test job name), give the action an `id`, and pass the four keystore inputs from secrets.
3. Set job `outputs` to `version_name` and `version_code` from `steps.<id>.outputs` so the deploy job can use them.
