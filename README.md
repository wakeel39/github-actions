# GitHub Actions

Reusable GitHub Actions used across our workflows. Each action lives in `.github/actions/` and can be referenced from any workflow in this repo (or others, if you use the repo path).

## Actions


| Action | Description |
|--------|-------------|
| [**build-android-appbundle**](.github/actions/build-android-appbundle/README.md) | Builds a signed Flutter Android AAB; sets version, keystore, key.properties, uploads artifact. |
| [**create-flutter-build**](.github/actions/create-flutter-build/README.md) | Flutter test: sets up Java/Flutter, pub get, doctor, analyzer, and tests for Flutter projects. |
| [**create-release-tag**](.github/actions/create-release-tag/README.md) | Creates a git tag and GitHub Release with deployment info (commit, author, message, changes). |
| [**deploy-to-playstore**](.github/actions/deploy-to-playstore/README.md) | Deploys an Android AAB to Google Play via Fastlane; sets up Java/Flutter/Ruby, runs Fastlane, optional release tag. |

## How to use an action

Reference an action from a workflow with `uses` and the path to the action folder:

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # needed if the action uses git history

      - uses: ./.github/actions/create-release-tag
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
```

For inputs and examples, see each action’s README in the table above.

## Repository structure

```
.github/
  actions/
    build-android-appbundle/
      action.yaml    # Action definition
      README.md      # Action documentation
    create-flutter-build/
      action.yaml    # Action definition
      README.md      # Action documentation
    create-release-tag/
      action.yaml    # Action definition
      README.md      # Action documentation
    deploy-to-playstore/
      action.yaml    # Action definition
      README.md      # Action documentation
README.md            # This file
```

## Adding or changing actions

- Add or edit the action in `.github/actions/<action-name>/`.
- Keep `action.yaml` as the main definition and document inputs, usage, and requirements in that folder’s `README.md`.
- Update this main README’s **Actions** table and any links when you add or rename actions.
