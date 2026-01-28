# Smart Dependabot Notifications

A GitHub Action that notifies developers only when dependabot pull requests fail checks and cannot be automerged. This prevents notification spam when dependabot pull requests are successfully automerged.

## Features

- ✅ Only triggers when workflow runs fail
- ✅ Only acts on dependabot PRs
- ✅ Supports both individual reviewers and team reviewers
- ✅ Customizable comment message

## Usage

### Basic Example

Create a workflow file (e.g., `.github/workflows/smart-dependabot-notifications.yml`) that triggers on workflow run completion:

```yaml
name: Smart Dependabot Notifications

on:
  workflow_run:
    workflows: ['CI', 'Tests']  # List your CI workflow names here
    types:
      - completed

jobs:
  notify-on-failure:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      actions: read
    steps:
      - name: Add reviewers on failure
        uses: dariacm/smart-dependabot-notifications@v1
        with:
          github-token: ${{ github.token }}
          reviewers: 'daria,carlotta'
          team-reviewers: 'backend-team,data-team'
```

### Advanced Example

```yaml
name: Smart Dependabot Notifications

on:
  workflow_run:
    workflows: ['CI', 'Lint', 'Tests']
    types:
      - completed

jobs:
  notify-on-failure:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      actions: read
    steps:
      - name: Add reviewers on failure
        uses: dariacm/smart-dependabot-notifications@v1
        with:
          github-token: ${{ github.token }}
          reviewers: 'daria,carlotta'
          team-reviewers: 'backend-team,data-team'
          comment: '🚨 Dependabot PR failed checks. Please review!'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token with `pull-requests:write` permission. Pass `${{ github.token }}` to use the default GitHub Actions token. | Yes | - |
| `reviewers` | Comma-separated list of GitHub usernames | No | - |
| `team-reviewers` | Comma-separated list of GitHub team slugs | No | - |
| `comment` | Custom comment to post on PR when checks fail | No | `dependabot was not able to automerge this pull request. Humans, please help 🤖` |

## Outputs

| Output | Description |
|--------|-------------|
| `pull-request-number` | The number of the pull request that was processed |
| `reviewers-added` | The total number of reviewers (users + teams) that were added |

## How It Works

1. **Triggers on workflow_run completion**: The action listens for completed workflow runs
2. **Checks actor**: Verifies the workflow run was triggered by `dependabot[bot]`
3. **Checks conclusion**: Only proceeds if the workflow run `conclusion` is `failure`
4. **Finds associated PR**: Gets the pull request associated with the workflow run
5. **Checks existing reviews**: Skips if reviewers are already requested
6. **Adds reviewers**: Requests reviews from specified users and/or teams
7. **Posts comment**: Adds a comment to notify reviewers (prevents duplicates)

## Permissions

The action requires the following permissions:

```yaml
permissions:
  pull-requests: write  # To request reviewers and post comments
  actions: read         # To read workflow run information
```

## Notes

- The action only works with `workflow_run` events
- At least one of `reviewers` or `team-reviewers` must be provided
- The action will skip if reviewers are already requested to avoid spam
- Comments are deduplicated by exact body match
- If no PR is associated with the workflow run, the action will exit gracefully

## License

This project is distributed under the [MIT license](LICENSE).