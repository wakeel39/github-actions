# Create Release Tag

Creates a GitHub release tag and release with deployment information. Use this action when you want to tag a commit and publish a GitHub Release with auto-generated release notes (deployment info, commit message, and changes).

## What it does

- Creates a **git tag** on the current (or specified) commit
- Pushes the tag to the repository
- Creates a **GitHub Release** for that tag with:
  - Deployment timestamp
  - Commit SHA, author, and message
  - Summary of changes from the commit

If you don’t provide a tag name, it generates one in the form: `vYYYY.MM.DD-YYYYMMDDHHMMSS`.

## Inputs

| Input         | Required | Description |
|---------------|----------|-------------|
| `github_token` | Yes     | GitHub token with permission to create releases (e.g. `${{ secrets.GITHUB_TOKEN }}`) |
| `repository`   | Yes     | Repository in `owner/repo` format |
| `tag_name`     | No      | Custom tag name. If omitted, a timestamp-based name is generated |
| `commit_sha`   | No      | Commit to tag. Defaults to current HEAD |
| `commit_msg`   | No      | Commit message for release notes. Defaults to the commit message from git |
| `commit_author`| No      | Author for release notes. Defaults to the commit author from git |

## Usage

### Basic (use defaults)

```yaml
- uses: ./.github/actions/create-release-tag
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
```

### With a custom tag name

```yaml
- uses: ./.github/actions/create-release-tag
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    tag_name: v1.2.3
```

### With explicit commit info (e.g. from another job)

```yaml
- uses: ./.github/actions/create-release-tag
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    tag_name: release-${{ github.run_id }}
    commit_sha: ${{ github.sha }}
    commit_msg: ${{ github.event.head_commit.message }}
    commit_author: ${{ github.event.head_commit.author.name }}
```

## Requirements

- **Bash** and **Node.js** (for the GitHub API call) must be available in the runner (default GitHub-hosted runners have both).
- The `github_token` must have `contents: write` so it can create releases and push tags. If the commit being tagged contains `.github/workflows/*.yml`, the job must also grant `workflows: write`, or the push will fail with *"refusing to allow a GitHub App to create or update workflow ... without `workflows` permission"*.

## Notes

- The action runs as a **composite action** and executes directly on the runner (no Docker).
- It creates a temporary `release_notes.md` and removes it after the release is created.
- Tag and release are created in the repository given by `repository`; use the same repo or a token that can write to the target repo.
