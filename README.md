# MorePET GitHub Actions

Centralized reusable GitHub Actions workflows for all MorePET repositories.

## Available Workflows

### 1. Renovate (Dependency Updates)

**File:** `.github/workflows/renovate-reusable.yml`

Automated dependency updates with configurable soak time.

**Usage in your repo:**

Create `.github/workflows/renovate.yml`:

```yaml
---
name: Renovate

on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly on Monday 2 AM UTC
  workflow_dispatch:

jobs:
  renovate:
    uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@main
    secrets:
      token: ${{ secrets.RENOVATE_TOKEN }}
```

**With custom config:**

```yaml
jobs:
  renovate:
    uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@main
    with:
      config_file: '.github/renovate.json'
      log_level: 'debug'
    secrets:
      token: ${{ secrets.RENOVATE_TOKEN }}
```

**Required:**
- Repository secret: `RENOVATE_TOKEN` (fine-grained PAT)
- Config file: `.github/renovate.json`

### 2. CI for Python Projects

**File:** `.github/workflows/ci-python-reusable.yml`

Runs pre-commit hooks and pytest for Python projects.

**Usage in your repo:**

Create `.github/workflows/ci.yml`:

```yaml
---
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    uses: MorePET/github-actions/.github/workflows/ci-python-reusable.yml@main
```

**With custom Python version:**

```yaml
jobs:
  test:
    uses: MorePET/github-actions/.github/workflows/ci-python-reusable.yml@main
    with:
      python_version: '3.11'
      run_tests: true
```

## Setup for New Repository

### 1. Create Renovate Token (Once per repo)

1. Go to: [github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta)
2. Click "Generate new token"
3. Configure:
   - Name: `Renovate Self-Hosted`
   - Expiration: 90 days
   - Repository access: Select your repository
   - Permissions:
     - Contents: Read and write
     - Pull requests: Read and write
     - Workflows: Read and write
4. Copy token and add as repository secret: `RENOVATE_TOKEN`

### 2. Add Workflows to Your Repo

Create these files:

**`.github/workflows/renovate.yml`:**

```yaml
---
name: Renovate

on:
  schedule:
    - cron: '0 2 * * 1'
  workflow_dispatch:

jobs:
  renovate:
    uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@main
    secrets:
      token: ${{ secrets.RENOVATE_TOKEN }}
```

**`.github/workflows/ci.yml`:**

```yaml
---
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    uses: MorePET/github-actions/.github/workflows/ci-python-reusable.yml@main
```

**`.github/renovate.json`:**

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "packageRules": [
    {
      "description": "Security updates - immediate",
      "matchManagers": ["pep621"],
      "vulnerabilityAlerts": {"enabled": true},
      "automerge": true,
      "automergeType": "pr",
      "minimumReleaseAge": null,
      "schedule": ["at any time"],
      "labels": ["dependencies", "security", "automerge"]
    },
    {
      "description": "Patch updates - 3 day soak time",
      "matchUpdateTypes": ["patch"],
      "matchManagers": ["pep621"],
      "groupName": "Python patch updates",
      "automerge": true,
      "automergeType": "pr",
      "minimumReleaseAge": "3 days",
      "schedule": ["at any time"],
      "labels": ["dependencies", "patch", "automerge"]
    },
    {
      "description": "Minor updates - manual review",
      "matchUpdateTypes": ["minor"],
      "matchManagers": ["pep621"],
      "automerge": false,
      "schedule": ["every weekend"],
      "labels": ["dependencies", "minor"]
    },
    {
      "description": "Major updates - manual review",
      "matchUpdateTypes": ["major"],
      "matchManagers": ["pep621"],
      "automerge": false,
      "schedule": ["every 2 weeks on Monday"],
      "labels": ["dependencies", "major"]
    }
  ],
  "prConcurrentLimit": 5,
  "prHourlyLimit": 2
}
```

### 3. Push to Main

```bash
git add .github/
git commit -m "ci: add centralized GitHub Actions workflows"
git push origin main
```

### 4. Test

1. Go to Actions tab
2. Select "Renovate" workflow
3. Click "Run workflow"
4. Verify it runs successfully

## Benefits

- ✅ **One update updates all repos** - Update workflow here, all repos get it
- ✅ **Consistent standards** - Same CI/CD across all MorePET repos
- ✅ **Less duplication** - Each repo has small caller workflows
- ✅ **Centralized maintenance** - Fix bugs in one place
- ✅ **Version control** - Use `@main` or pin to `@v1.0.0`

## Versioning

**Use `@main` for latest:**
```yaml
uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@main
```

**Pin to version (recommended for production):**
```yaml
uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@v1.0.0
```

## Updating These Workflows

1. Update workflow in this repo
2. Commit and push to `main`
3. All repos using `@main` automatically get the update
4. Tag releases for stability: `git tag -a v1.1.0 -m "Release v1.1.0"`

## Troubleshooting

### Workflow not found error

→ Ensure workflow file exists on `main` branch in this repo

### Permission denied

→ Check that calling repo has access (should if both in MorePET org)

### Token issues

→ Verify `RENOVATE_TOKEN` secret exists in calling repository

## Support

For issues or questions, open an issue in this repository.

