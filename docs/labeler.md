# Pull Request Labeler Workflow

## Overview

The Pull Request Labeler is a reusable GitHub Actions workflow that automatically applies labels to pull requests based on the files that have been changed. This workflow helps maintain consistency in PR labeling across your organization and reduces manual effort.

## Features

- 🏷️ Automatically labels PRs based on changed file patterns
- 🔄 Optional label sync (removes labels when matching files are reverted)
- 🌐 Supports both local and centralized configuration
- ⚙️ Fully customizable label rules

## Usage

### Basic Usage (Local Configuration)

Create a workflow file in your repository (e.g., `.github/workflows/pr-labeler.yml`):

```yaml
name: "PR Labeling"

on:
  pull_request_target:

jobs:
  labeler:
    uses: san-ye-zi/github-actions/.github/workflows/labeler.yml@main
    with:
      config-path: '.github/labeler.yml'
```

### Advanced Usage (Centralized Configuration)

Use a shared configuration from a central repository:

```yaml
name: "PR Labeling"

on:
  pull_request_target:

jobs:
  labeler:
    uses: san-ye-zi/github-actions/.github/workflows/labeler.yml@main
    with:
      config-path: '.github/labeler.yml'
      config-repo: 'my-org/central-configs'
      sync-labels: true
```

## Input Parameters

| Parameter | Description | Required | Default | Type |
|-----------|-------------|----------|---------|------|
| `config-repo` | Organization/repository for central configs (leave empty for local) | No | `''` | string |
| `config-path` | Path to the labeler configuration file | No | `.github/labeler.yml` | string |
| `sync-labels` | Whether to remove labels when matching files are reverted | No | `true` | boolean |

## Configuration File Format

Create a labeler configuration file (e.g., `.github/labeler.yml`) with the following format:

```yaml
# Label name
'label-name':
  - changed-files:
    - any-glob-to-any-file: 'path/to/files/**/*'

# Multiple patterns for the same label
'documentation':
  - changed-files:
    - any-glob-to-any-file:
      - '**/*.md'
      - 'docs/**/*'

# Platform-specific labels
'platform: android':
  - changed-files:
    - any-glob-to-any-file: 'android/**/*'

'platform: ios':
  - changed-files:
    - any-glob-to-any-file: 'ios/**/*'
```

### Example Configuration

See the included [flutter-labeler.yml](../shared-config/flutter-labeler.yml) for a complete example that demonstrates:
- Platform-specific labels (Android, iOS, Web)
- UI component labels
- Localization labels
- Test file labels
- CI/CD labels
- Dependency labels
- Documentation labels

## How It Works

1. **Trigger**: The workflow runs when a pull request is opened or updated
2. **Configuration Loading**: 
   - If `config-repo` is provided, it checks out that repository to access the configuration
   - Otherwise, it uses the configuration from the current repository
3. **Label Application**: The labeler action reads the configuration and applies matching labels based on changed files
4. **Label Sync**: If enabled, labels are removed when the matching files are no longer part of the PR changes

## Permissions Required

The workflow requires the following permissions:

```yaml
permissions:
  contents: read        # To read repository contents
  pull-requests: write  # To add/remove labels on PRs
```

These permissions are automatically set within the workflow.

## Best Practices

1. **Use `pull_request_target`**: This event is recommended for labeler workflows as it runs with repository permissions even for PRs from forks
2. **Centralize configurations**: For organizations with multiple repositories, use a central config repository to maintain consistency
3. **Keep patterns specific**: Use precise glob patterns to avoid over-labeling
4. **Enable sync-labels**: Keep this enabled to automatically remove labels when files are reverted
5. **Regular reviews**: Periodically review and update your labeling rules to match your team's needs

## Troubleshooting

### Labels not being applied

- Verify your configuration file path is correct
- Check that the glob patterns match your file structure
- Ensure the workflow has proper permissions

### Labels not being removed

- Check that `sync-labels` is set to `true`
- Verify the PR has been updated to remove the files that triggered the label

### Central configuration not found

- Verify the `config-repo` format is `organization/repository`
- Ensure the specified path exists in the central repository
- Check that the repository is accessible (public or has proper access)

## Related Resources

- [actions/labeler documentation](https://github.com/actions/labeler)
- [GitHub Actions workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Pull request events](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target)
