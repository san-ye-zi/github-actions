# Flutter Localization Workflow

## Overview

The Flutter Localization workflow is a reusable GitHub Actions workflow that validates localization (l10n) files in Flutter projects. It ensures that generated l10n files are always up-to-date with the source `.arb` files, preventing outdated translations from being merged into your codebase.

## Features

- ✅ Validates that l10n files are up-to-date
- 🔄 Automatically generates l10n files for verification
- 📊 Provides detailed output showing what changed
- ⚙️ Configurable Flutter version and channel
- 🎯 Optional failure on outdated files
- 📤 Status output for downstream workflows

## Usage

### Basic Usage

Create a workflow file in your repository (e.g., `.github/workflows/l10n-check.yml`):

```yaml
name: Localization Check

on:
  pull_request:
    branches:
      - main
    paths:
      - 'lib/l10n/**'
      - '**/l10n.yaml'

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      fail-on-changes: true
```

### Advanced Usage

Use custom Flutter version and command:

```yaml
name: Localization Check

on:
  pull_request:
    paths:
      - 'lib/l10n/**'
      - '**/*.arb'

jobs:
  localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      flutter-version: '3.38.5'
      flutter-channel: 'stable'
      working-directory: './app'
      fail-on-changes: true
      l10n-command: 'flutter gen-l10n --template-arb-file=app_en.arb'
```

### Using Workflow Outputs

Chain workflows based on l10n validation status:

```yaml
jobs:
  localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      fail-on-changes: false

  notify:
    needs: localization
    runs-on: ubuntu-latest
    if: needs.localization.outputs.l10n-status == 'outdated'
    steps:
      - name: Post comment
        run: |
          echo "L10n files need to be regenerated!"
```

## Input Parameters

| Parameter | Description | Required | Default | Type |
|-----------|-------------|----------|---------|------|
| `flutter-version` | Flutter version to use | No | `3.38.5` | string |
| `flutter-channel` | Flutter channel (stable, beta, dev) | No | `stable` | string |
| `runs-on` | Runner to use for the job | No | `ubuntu-latest` | string |
| `working-directory` | Working directory for Flutter commands | No | `.` | string |
| `fail-on-changes` | Fail if l10n files are not up to date | No | `true` | boolean |
| `l10n-command` | Custom l10n generation command | No | `flutter gen-l10n` | string |

## Output Parameters

| Output | Description | Values |
|--------|-------------|--------|
| `l10n-status` | Status of l10n validation | `up-to-date` or `outdated` |

## How It Works

1. **Checkout**: Checks out the repository code
2. **Setup Flutter**: Installs the specified Flutter version and channel
3. **Get Dependencies**: Runs `flutter pub get` to fetch project dependencies
4. **Generate L10n**: Executes the l10n generation command
5. **Check Changes**: Compares generated files with committed files
6. **Report Status**: 
   - If files match: ✅ Workflow succeeds
   - If files differ: ❌ Shows diff and fails (if `fail-on-changes: true`)

## Configuration Requirements

Your Flutter project should have a proper l10n configuration. Create or update `l10n.yaml` in your project root:

```yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
```

## Example Project Structure

```
my_flutter_app/
├── lib/
│   ├── l10n/
│   │   ├── app_en.arb          # Template file
│   │   ├── app_es.arb          # Spanish translations
│   │   ├── app_fr.arb          # French translations
│   │   └── l10n.dart           # Generated file
│   └── main.dart
├── l10n.yaml                    # L10n configuration
└── pubspec.yaml
```

## Common Use Cases

### 1. Enforce Updated Translations on PRs

Ensure developers update l10n files when modifying `.arb` files:

```yaml
on:
  pull_request:
    paths:
      - '**/*.arb'
      - 'lib/l10n/**'

jobs:
  localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      fail-on-changes: true
```

### 2. Warning-Only Mode

Check l10n status without failing the workflow:

```yaml
jobs:
  localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      fail-on-changes: false
```

### 3. Monorepo with Multiple Flutter Apps

Validate l10n in a specific subdirectory:

```yaml
jobs:
  mobile-app-localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      working-directory: './apps/mobile'
      
  web-app-localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      working-directory: './apps/web'
```

### 4. Custom Generation Command

Use custom parameters for l10n generation:

```yaml
jobs:
  localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    with:
      l10n-command: 'flutter gen-l10n --template-arb-file=intl_en.arb --output-dir=lib/generated'
```

## Best Practices

1. **Trigger on relevant paths**: Only run the workflow when l10n files change to save CI minutes
2. **Use concurrency controls**: Cancel outdated workflow runs when new commits are pushed
3. **Commit generated files**: Always commit generated l10n files to your repository
4. **Set fail-on-changes to true**: Enforce that developers regenerate files before merging
5. **Use stable channel**: Stick with stable Flutter releases for consistency unless you need beta features

## Troubleshooting

### Workflow fails with "l10n files are not up to date"

**Solution**: Run the l10n generation command locally and commit the changes:

```bash
flutter gen-l10n
git add .
git commit -m "Update l10n files"
git push
```

### Generated files differ between local and CI

**Possible causes**:
- Different Flutter versions (specify exact version in workflow)
- Different l10n configuration (ensure `l10n.yaml` is committed)
- Line ending differences (configure `.gitattributes` properly)

**Solution**: Specify the exact Flutter version:

```yaml
with:
  flutter-version: '3.38.5'
  flutter-channel: 'stable'
```

### Workflow doesn't run

**Check**:
- The `paths` filter matches your l10n file locations
- The branch filter includes your target branch
- The workflow file is in `.github/workflows/` directory

### Custom l10n command not working

**Solution**: Verify the command works locally first:

```bash
cd your-working-directory
flutter gen-l10n --your-custom-flags
```

Then use that exact command in the workflow:

```yaml
with:
  l10n-command: 'flutter gen-l10n --your-custom-flags'
  working-directory: 'your-working-directory'
```

## Integration with Other Workflows

### Pre-commit Hook

Add a local check before committing:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: flutter-l10n
        name: Generate Flutter l10n
        entry: flutter gen-l10n
        language: system
        pass_filenames: false
```

### Combined CI Pipeline

```yaml
jobs:
  test:
    uses: ./.github/workflows/test.yml
    
  localization:
    uses: san-ye-zi/github-actions/.github/workflows/flutter-localization.yml@main
    
  analyze:
    needs: [test, localization]
    uses: ./.github/workflows/analyze.yml
```

## Related Resources

- [Flutter Internationalization Guide](https://docs.flutter.dev/development/accessibility-and-localization/internationalization)
- [ARB File Format](https://github.com/google/app-resource-bundle/wiki/ApplicationResourceBundleSpecification)
- [Flutter gen-l10n Command](https://docs.flutter.dev/development/accessibility-and-localization/internationalization#configuring-the-l10n-yaml-file)
- [GitHub Actions Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
