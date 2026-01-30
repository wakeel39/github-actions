# Deploy to Play Store

Deploys an Android App Bundle (AAB) to Google Play using **Fastlane**. Use this action in a workflow after a build job that uploads the AAB as an artifact. It sets up Java, Flutter, Ruby, installs Fastlane, downloads the AAB, validates the service account JSON, determines the release track, runs Fastlane deploy, optionally creates a GitHub release tag, and cleans up secrets.

## What it does

1. **Checkout** – Checks out your repo
2. **Setup** – Java (Temurin), Flutter, Ruby, and Fastlane (`bundle install` in your Fastlane directory)
3. **Download AAB** – Downloads the artifact that contains the AAB (default name: `app-bundle`)
4. **Verify AAB** – Ensures `app-release.aab` exists at the expected path
5. **Service account** – Writes and validates `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` to `google-play-service-account.json`
6. **Track** – Chooses Play track: `internal`, `alpha`, `beta`, or `production` (see below)
7. **Deploy** – Runs `bundle exec fastlane android deploy track:<track>`
8. **Release tag** (optional) – Creates a GitHub release tag using `create-release-tag`
9. **Cleanup** – Removes service account file, `key.properties`, and `keystore.jks`

### How the track is chosen

| Condition | Track |
|-----------|--------|
| You pass `track` input | That value |
| `workflow_dispatch` and workflow input `track` is set | That value |
| Ref is a tag like `refs/tags/v*` | `production` |
| Otherwise (e.g. `main`/`master`) | `internal` |

## Prerequisites in your repo

- **Fastlane** in a directory (default: `fastlane`) with:
  - `Gemfile` and `bundle install`-able dependencies
  - Lane: `android deploy` that accepts `track:...` and uses the service account (e.g. from `google-play-service-account.json` in the repo root or via env)
- **Build job** that produces an AAB and uploads it with `actions/upload-artifact` (default artifact name: `app-bundle`).
- **Secrets**: `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` – full JSON key for the Google Play service account.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `google_play_service_account_json` | No* | - | JSON content of the Google Play service account (use a secret). *Or use `service_account_json`. |
| `service_account_json` | No* | - | Alias for `google_play_service_account_json` (pass one or the other). |
| `aab_artifact_name` | No | `app-bundle` | Name of the artifact that contains the AAB. |
| `aab_path` | No | `build/app/outputs/bundle/release/` | Path where the artifact is extracted; AAB is expected at `<aab_path>/app-release.aab`. |
| `version_name` | No | - | App version (e.g. `1.2.3`). Used for release tag when `create_release_tag` is true (tag becomes `v{version_name}` unless `tag_name` is set). |
| `version_code` | No | - | App version code (optional; accepted but not used by deploy). |
| `tag_name` | No | - | Override release tag name (default is `v` + `version_name`). |
| `track` | No | (auto) | Override Play track: `internal`, `alpha`, `beta`, `production`. |
| `flutter_version` | No | `3.24.0` | Flutter version for setup. |
| `java_version` | No | `17` | Java version for setup. |
| `ruby_version` | No | `3.2` | Ruby version for Fastlane. |
| `fastlane_directory` | No | `fastlane` | Directory containing Fastlane (Gemfile, lanes). |
| `create_release_tag` | No | `true` | Set to `false` to skip creating a GitHub release tag. |
| `github_token` | No | - | Token for creating the release tag (e.g. `${{ github.token }}`). Required when `create_release_tag` is true. |
| `repository` | No | - | Repo in `owner/repo` for the release tag (e.g. `${{ github.repository }}`). Required when `create_release_tag` is true. |

## Usage

### 1. In the same repo (local action)

```yaml
deploy:
  name: Deploy to Play Store
  needs: build
  runs-on: ubuntu-latest
  if: github.event_name == 'workflow_dispatch' || startsWith(github.ref, 'refs/tags/v') || github.ref == 'refs/heads/main'
  permissions:
    contents: write
  steps:
    - uses: ./.github/actions/deploy-to-playstore
      with:
        google_play_service_account_json: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
        version_name: ${{ needs.build.outputs.version_name }}
```

### 2. From this GitHub Actions repo (another repo)

```yaml
deploy:
  name: Deploy to Play Store
  needs: build
  runs-on: ubuntu-latest
  if: github.event_name == 'workflow_dispatch' || startsWith(github.ref, 'refs/tags/v') || github.ref == 'refs/heads/main'
  permissions:
    contents: write
  steps:
    - uses: wakeel39/github-actions/.github/actions/deploy-to-playstore@main
      with:
        google_play_service_account_json: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
        version_name: ${{ needs.build.outputs.version_name }}
```

### 3. With manual track (workflow_dispatch)

In your workflow, add an input and pass it to the action:

```yaml
on:
  workflow_dispatch:
    inputs:
      track:
        description: 'Play Store track'
        required: true
        default: 'internal'
        type: choice
        options:
          - internal
          - alpha
          - beta
          - production

jobs:
  deploy:
    steps:
      - uses: wakeel39/github-actions/.github/actions/deploy-to-playstore@main
        with:
          google_play_service_account_json: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
          version_name: ${{ needs.build.outputs.version_name }}
          track: ${{ github.event.inputs.track }}
```

### 4. Custom artifact name and path

If your build job uploads the AAB with a different name or path:

```yaml
- uses: ./.github/actions/deploy-to-playstore
  with:
    google_play_service_account_json: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
    aab_artifact_name: my-app-aab
    aab_path: app/build/outputs/bundle/release/
    version_name: ${{ needs.build.outputs.version_name }}
```

### 5. Skip release tag

```yaml
- uses: ./.github/actions/deploy-to-playstore
  with:
    google_play_service_account_json: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
    create_release_tag: 'false'
```

## Build job: artifact name and path

Your build job must upload the AAB so this action can download it and find `app-release.aab`:

```yaml
build:
  runs-on: ubuntu-latest
  outputs:
    version_name: ${{ steps.version.outputs.version_name }}
  steps:
    # ... build steps that produce build/app/outputs/bundle/release/app-release.aab ...
    - uses: actions/upload-artifact@v4
      with:
        name: app-bundle
        path: build/app/outputs/bundle/release/
```

If your path or artifact name is different, set `aab_artifact_name` and `aab_path` accordingly.

## Fastlane

- Put Fastlane in the directory specified by `fastlane_directory` (default: `fastlane`).
- The action runs: `bundle exec fastlane android deploy track:<track>` from that directory.
- Ensure your lane reads the service account from the repo root (e.g. `google-play-service-account.json`) or from `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`; the action sets the env var and writes the file to the workspace root.

## Requirements

- **Runner**: Ubuntu (e.g. `ubuntu-latest`); the action uses Bash and Python 3.
- **Secrets**: `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` in your repo/organization secrets.
- **Permissions**: For **release tags to work**, the deploy job must have `contents: write` so the token can push the tag and create the release. Add this to the job that uses this action:

  ```yaml
  deploy:
    permissions:
      contents: write
    steps:
      - uses: ...
  ```

## Using in your own repo

1. In your app repo, add a workflow (e.g. `.github/workflows/deploy.yml`) that has a `build` job and a `deploy` job.
2. In `deploy`, use either:
   - **Same repo**: `uses: ./.github/actions/deploy-to-playstore` (after copying this action into your repo), or
   - **This repo**: `uses: wakeel39/github-actions/.github/actions/deploy-to-playstore@main`
3. Pass `google_play_service_account_json` (and optionally `version_name`, `track`, etc.) as in the examples above.
4. Ensure the build job uploads the AAB artifact and that Fastlane is set up as described.
