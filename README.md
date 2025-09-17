# Renovate [shareable config](https://renovatebot.com/docs/config-presets/) for [SweetAlert2](https://github.com/sweetalert2/sweetalert2)

## Usage

Enable Renovate in your repo and just `extends` in `renovate.json`.

```js
{
  "extends": ["github>sweetalert2/renovate-config"]
  // Additional, per-project rules...
}
```

Note: You don't have to install this config. Renovate fetches it from this GitHub repo automatically.

## TODO

- rollup minor and patch updates on a weekly basis https://github.com/renovatebot/renovate/issues/4404

---

# Renovate Minimum Release Age Policy Checker

A reusable GitHub Actions workflow that enforces minimum release age policies for Renovate dependency updates.

## 🚀 Features

- **Automatic Policy Enforcement**: Validates that new package versions meet your minimum release age requirements
- **Flexible Configuration**: Supports multiple config sources and override options
- **Comprehensive Analysis**: Provides detailed version analysis and release date information
- **Rich Reporting**: Generates professional job summaries with policy compliance details
- **Reusable**: Can be shared across multiple repositories
- **Customizable**: Supports custom PR title patterns and configuration sources

## 📋 Quick Start

### Using in Your Repository

Create `.github/workflows/renovate-policy-check.yml` in your repository:

```yaml
name: Enforce Minimum Release Age Policy

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  check-renovate-policy:
    if: github.actor == 'renovate[bot]' || github.event.pull_request.user.login == 'renovate[bot]'
    uses: sweetalert2/renovate-config/.github/workflows/renovate-policy-check-reusable.yml@main
```

### With Custom Configuration

```yaml
jobs:
  check-renovate-policy:
    if: github.actor == 'renovate[bot]'
    uses: sweetalert2/renovate-config/.github/workflows/renovate-policy-check-reusable.yml@main
    with:
      # Use your own renovate config URL
      renovate-config-url: 'https://raw.githubusercontent.com/your-org/renovate-config/main/default.json'

      # Override minimum release age for this specific repo
      minimum-release-age-override: '7 days'

      # Custom PR title pattern if needed
      pr-title-pattern: 'chore\(deps\):\ update\ ([^\ ]+)\ to\ v(.+)'
```

## 🔧 Configuration Options

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `renovate-config-url` | URL to fetch renovate configuration from | No | `https://raw.githubusercontent.com/sweetalert2/renovate-config/main/default.json` |
| `minimum-release-age-override` | Override minimum release age (e.g., "14 days", "7d", "2w") | No | - |
| `pr-title-pattern` | Custom regex pattern for PR title parsing | No | `fix\(deps\):\ update\ dependency\ ([^\ ]+)\ to\ v(.+)` |

### Outputs

| Output | Description |
|--------|-------------|
| `package-name` | Extracted package name |
| `current-version` | Current version from PR |
| `next-version` | Next available version |
| `days-difference` | Days between current and next version release |
| `policy-compliant` | Whether the update complies with policy (`true`, `false`, or `skipped`) |

## 📊 What It Does

1. **Extracts Package Information**: Parses Renovate PR titles to identify package and version
2. **Analyzes Version History**: Fetches all available versions and determines the next version
3. **Calculates Release Dates**: Determines when each version was published
4. **Enforces Policy**: Compares release age difference against your minimum release age requirement
5. **Reports Results**: Provides comprehensive job summaries and policy compliance status

## 🎯 Example Use Cases

### Basic Policy Enforcement
```yaml
# Fails the workflow if policy is violated
uses: sweetalert2/renovate-config/.github/workflows/renovate-policy-check-reusable.yml@main
```

### Advanced Workflow with Actions
```yaml
jobs:
  policy-check:
    uses: sweetalert2/renovate-config/.github/workflows/renovate-policy-check-reusable.yml@main

  handle-results:
    needs: policy-check
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Auto-approve Compliant PRs
        if: needs.policy-check.outputs.policy-compliant == 'true'
        uses: hmarr/auto-approve-action@v3

      - name: Add Warning Comment
        if: needs.policy-check.outputs.policy-compliant == 'false'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '⚠️ This PR violates the minimum release age policy and requires manual review.'
            })
```

## 🔍 Supported Configuration Formats

The workflow supports multiple ways to specify `minimumReleaseAge`:

- `"14 days"` - Explicit days with text
- `"7d"` - Short day format
- `"2 weeks"` - Weeks format
- `"1w"` - Short week format
- `"14"` - Just a number (assumes days)

## 📁 Configuration Priority

The workflow checks for configuration in this order:

1. **Input Override**: `minimum-release-age-override` parameter
2. **Remote Config**: URL specified in `renovate-config-url`
3. **Local Config**: `renovate.json` in the repository
4. **Skip**: If no configuration is found, policy check is skipped

## 🎨 Job Summary

The workflow generates a comprehensive job summary that includes:

- **PR Information**: Title, author, number, repository
- **Package Analysis**: Package name, versions, update type
- **Version Analysis**: Current version position, next available version, latest versions
- **Release Date Analysis**: Release dates, version ages, time differences
- **Policy Check Results**: Compliance status, configuration source, detailed policy analysis

## 🚀 Setting Up as a Reusable Workflow

### 1. Create Dedicated Repository

Create a new repository (e.g., `your-org/renovate-policy-workflows`) and add the reusable workflow file.

### 2. Make Repository Public or Grant Access

- **Public Repository**: Anyone can use the workflow
- **Private Repository**: Grant access to repositories that need to use it

### 3. Reference in Other Repositories

```yaml
uses: your-org/renovate-policy-workflows/.github/workflows/renovate-policy-check.yml@main
```

### 4. Version Your Workflows

Use tags for stable releases:

```yaml
uses: your-org/renovate-policy-workflows/.github/workflows/renovate-policy-check.yml@v1.0.0
```

## 🔒 Security Considerations

- The workflow only runs on Renovate PRs (filtered by bot user)
- No secrets are required for basic functionality
- External config URLs should be from trusted sources
- Consider pinning to specific workflow versions in production

## 🤝 Contributing

Feel free to submit issues, feature requests, or pull requests to improve this reusable workflow!

## 📄 License

MIT License - feel free to use and modify as needed.
