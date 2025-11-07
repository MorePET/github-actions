# MorePET GitHub Actions

Centralized reusable GitHub Actions workflows for all MorePET repositories.

## Available Workflows

### 1. Renovate (Dependency Updates)

**File:** `.github/workflows/renovate-reusable.yml`

A single reusable workflow for all Renovate use cases. Consuming repos create multiple workflows calling this with different config files.

## Renovate Architecture

MorePET uses **three specialized Renovate workflows** in each consuming repo:

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `renovate-external-dependencies.yml` | Scheduled (weekly) | PyPI, pre-commit, GitHub Actions |
| `renovate-internal-dependencies.yml` | Triggered by releases | MorePET internal packages/tools |
| `renovate-internal-containers.yml` | Triggered by releases | MorePET container images |

All three call the same reusable workflow with different config files.

### 1a. External Dependencies (Scheduled)

**Purpose:** Updates from PyPI, pre-commit, GitHub Actions

**Create in your repo:** `.github/workflows/renovate-external-dependencies.yml`

```yaml
---
name: Renovate (External Dependencies)

on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly on Monday at 2 AM UTC
  workflow_dispatch:

jobs:
  renovate:
    uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@main
    with:
      config_file: '.github/renovate-external-dependencies.json'
    secrets:
      token: ${{ secrets.RENOVATE_TOKEN }}
```

**Config:** `.github/renovate-external-dependencies.json` (see templates below)

### 1b. Internal Dependencies (Triggered)

**Purpose:** Updates from MorePET internal packages/tools (triggered by package releases)

**Create in your repo:** `.github/workflows/renovate-internal-dependencies.yml`

```yaml
---
name: Renovate (Internal Dependencies)

on:
  workflow_dispatch:  # Triggered by internal MorePET package releases

jobs:
  renovate:
    uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@main
    with:
      config_file: '.github/renovate-internal-dependencies.json'
    secrets:
      token: ${{ secrets.RENOVATE_TOKEN }}
```

**Config:** `.github/renovate-internal-dependencies.json` (see templates below)

### 1c. Internal Containers (Triggered)

**Purpose:** Updates from MorePET container images (triggered by MorePET/containers releases)

**Create in your repo:** `.github/workflows/renovate-internal-containers.yml`

```yaml
---
name: Renovate (Internal Containers)

on:
  workflow_dispatch:  # Triggered by MorePET/containers releases

jobs:
  renovate:
    uses: MorePET/github-actions/.github/workflows/renovate-reusable.yml@main
    with:
      config_file: '.github/renovate-internal-containers.json'
    secrets:
      token: ${{ secrets.RENOVATE_TOKEN }}
```

**Config:** `.github/renovate-internal-containers.json` (see templates below)

**Monitored Container Types:**
- Devcontainers (`.devcontainer/devcontainer.json`)
- Docker/Podman (`Dockerfile`, `Containerfile`)
- Compose (`docker-compose.yml`, `podman-compose.yml`)

## Config File Templates

### External Dependencies Config

**File:** `.github/renovate-external-dependencies.json`

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "packageRules": [
    {
      "description": "Security patches - immediate",
      "matchUpdateTypes": ["patch"],
      "matchManagers": ["pep621"],
      "vulnerabilityAlerts": { "enabled": true },
      "automerge": true,
      "automergeType": "pr",
      "minimumReleaseAge": null,
      "schedule": ["at any time"],
      "labels": ["dependencies", "security", "patch", "automerge"]
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
      "groupName": "Python minor updates",
      "automerge": false,
      "schedule": ["every weekend"],
      "labels": ["dependencies", "minor"]
    },
    {
      "description": "Major updates - manual review",
      "matchUpdateTypes": ["major"],
      "matchManagers": ["pep621"],
      "groupName": "Python major updates",
      "automerge": false,
      "schedule": ["every 2 weeks on Monday"],
      "labels": ["dependencies", "major"]
    },
    {
      "description": "Pre-commit hooks",
      "matchManagers": ["pre-commit"],
      "groupName": "Pre-commit hooks",
      "automerge": false,
      "schedule": ["every 2 weeks on Monday"],
      "labels": ["dependencies", "pre-commit"]
    },
    {
      "description": "GitHub Actions",
      "matchManagers": ["github-actions"],
      "groupName": "GitHub Actions",
      "automerge": true,
      "automergeType": "pr",
      "schedule": ["every 2 weeks on Monday"],
      "labels": ["dependencies", "github-actions", "automerge"]
    }
  ],
  "prConcurrentLimit": 5,
  "prHourlyLimit": 2
}
```

### Internal Dependencies Config

**File:** `.github/renovate-internal-dependencies.json`

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "packageRules": [
    {
      "description": "Internal MorePET packages - security patches",
      "matchUpdateTypes": ["patch"],
      "matchManagers": ["pep621"],
      "matchPackagePatterns": ["^morepet-", "^MorePET/"],
      "vulnerabilityAlerts": { "enabled": true },
      "automerge": true,
      "automergeType": "pr",
      "minimumReleaseAge": null,
      "schedule": ["at any time"],
      "labels": ["dependencies", "internal", "security", "patch", "automerge"]
    },
    {
      "description": "Internal MorePET packages - patch updates",
      "matchUpdateTypes": ["patch"],
      "matchManagers": ["pep621"],
      "matchPackagePatterns": ["^morepet-", "^MorePET/"],
      "groupName": "MorePET internal patch updates",
      "automerge": true,
      "automergeType": "pr",
      "minimumReleaseAge": "3 days",
      "schedule": ["at any time"],
      "labels": ["dependencies", "internal", "patch", "automerge"]
    },
    {
      "description": "Internal MorePET packages - minor/major manual review",
      "matchUpdateTypes": ["minor", "major"],
      "matchManagers": ["pep621"],
      "matchPackagePatterns": ["^morepet-", "^MorePET/"],
      "automerge": false,
      "schedule": ["at any time"],
      "labels": ["dependencies", "internal"]
    }
  ],
  "hostRules": [
    {
      "matchHost": "https://github.com",
      "hostType": "github-packages"
    }
  ],
  "prConcurrentLimit": 5,
  "prHourlyLimit": 2
}
```

### Internal Containers Config

**File:** `.github/renovate-internal-containers.json`

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "packageRules": [
    {
      "description": "MorePET containers - all types (Docker, Podman, devcontainer, compose)",
      "matchManagers": ["devcontainer", "docker", "docker-compose"],
      "matchPackagePatterns": ["ghcr.io/morepet/containers/"],
      "automerge": false,
      "schedule": ["at any time"],
      "labels": ["dependencies", "internal", "container"]
    }
  ],
  "prConcurrentLimit": 5,
  "prHourlyLimit": 2
}
```

## Auto-Discovery Pattern

Internal dependencies and containers use **auto-discovery** to trigger consuming repos:

### How It Works

1. **Producer repo releases** (e.g., MorePET/containers tags new image)
2. **Search for consumers** via GitHub Code Search API
3. **Find all repos** with the corresponding workflow file
4. **Trigger workflows** automatically in those repos
5. **Create PRs** within minutes of release

### Implementation in Producer Repos

Add to your release workflow (in MorePET/containers or internal package repos):

```yaml
name: Release

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      # Your build/release steps here
      
      - name: Trigger Renovate in consuming repos
        run: |
          # For containers: renovate-internal-containers.yml
          # For packages: renovate-internal-dependencies.yml
          WORKFLOW_FILE="renovate-internal-containers.yml"
          
          echo "Finding repos with $WORKFLOW_FILE..."
          
          gh api "/search/code?q=path:.github/workflows/${WORKFLOW_FILE}+org:MorePET" \
            --jq '.items[].repository.name' | \
          while read -r repo; do
            echo "Triggering $WORKFLOW_FILE in MorePET/$repo..."
            gh workflow run "$WORKFLOW_FILE" \
              --repo "MorePET/$repo" \
              --ref main || echo "Failed to trigger $repo"
          done
        env:
          GH_TOKEN: ${{ secrets.ORG_PAT }}
```

**Required in Producer Repos:**
- Organization-wide PAT with `actions:write` permission
- Add as secret: `ORG_PAT`
- Use semantic versioning for releases

### Benefits

- **Zero maintenance** - No list of consuming repos to update
- **Self-service** - Repos opt-in by adding the workflow
- **Automatic discovery** - New consumers automatically included
- **Loose coupling** - Repos discover each other via workflow presence

## Auto-merge Behavior

### External Dependencies

| Update Type | Soak Time | Auto-merge? | Labels |
|-------------|-----------|-------------|--------|
| **Security Patch** | Immediate | ✅ Yes | `security`, `patch`, `automerge` |
| **Patch** (x.x.N) | 3 days | ✅ Yes | `patch`, `automerge` |
| **Minor** (x.N.0) | - | ❌ No (manual review) | `minor` |
| **Major** (N.0.0) | - | ❌ No (manual review) | `major` |

### Internal Dependencies

Same strategy as external, applies to MorePET internal packages:

| Update Type | Soak Time | Auto-merge? | Labels |
|-------------|-----------|-------------|--------|
| **Security Patch** | Immediate | ✅ Yes | `internal`, `security`, `patch`, `automerge` |
| **Patch** (x.x.N) | 3 days | ✅ Yes | `internal`, `patch`, `automerge` |
| **Minor** (x.N.0) | - | ❌ No (manual review) | `internal`, `minor` |
| **Major** (N.0.0) | - | ❌ No (manual review) | `internal`, `major` |

### Internal Containers

All container updates require manual review:

| Update Type | Auto-merge? | Labels |
|-------------|-------------|--------|
| **All updates** | ❌ No (manual review) | `internal`, `container` |

## CI/CD (Python Testing)

**File:** `.github/workflows/ci-python-reusable.yml`

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

## Setup Checklist

### For All Repos

- [ ] Copy `.github/workflows/ci.yml`
- [ ] Create fine-grained PAT with permissions:
  - Contents: Read and write
  - Pull requests: Read and write
  - Workflows: Read and write
- [ ] Add PAT as repository secret: `RENOVATE_TOKEN`
- [ ] Enable auto-merge in repository settings (Settings → General → Pull Requests)
- [ ] Push to main branch

### If Consuming External Dependencies

- [ ] Copy `.github/workflows/renovate-external-dependencies.yml`
- [ ] Copy `.github/renovate-external-dependencies.json`
- [ ] Manually trigger workflow to test

### If Consuming MorePET Internal Packages

- [ ] Copy `.github/workflows/renovate-internal-dependencies.yml`
- [ ] Copy `.github/renovate-internal-dependencies.json`
- [ ] Ensure producer repos have trigger logic

### If Consuming MorePET Containers

- [ ] Copy `.github/workflows/renovate-internal-containers.yml`
- [ ] Copy `.github/renovate-internal-containers.json`
- [ ] Ensure MorePET/containers has trigger logic

### If Producing Internal Packages/Containers

- [ ] Add all above (internal dependencies or containers)
- [ ] Add release workflow with auto-discovery trigger
- [ ] Create organization PAT: `ORG_PAT`
- [ ] Use semantic versioning for releases (`v1.2.3`)

## GitHub Hygiene for Internal Dependencies

For auto-discovery to work with internal packages:

- ✅ Use semantic versioning (`v1.2.3`)
- ✅ Create GitHub releases or tags
- ✅ Follow semver conventions (MAJOR.MINOR.PATCH)
- ✅ Add trigger logic to release workflow

## Required Secrets

### Repository Secrets (All Repos)

| Secret | Purpose | Permissions |
|--------|---------|-------------|
| `RENOVATE_TOKEN` | Renovate automation | Contents, PRs, Workflows (Read + Write) |

### Organization Secrets (Producer Repos Only)

| Secret | Purpose | Permissions |
|--------|---------|-------------|
| `ORG_PAT` | Trigger workflows in other repos | Actions (Write) |

## Troubleshooting

### External Renovate creates no PRs

→ Check workflow logs, may indicate:
- No updates available
- Configuration error
- Token permissions issue

### Internal workflows not triggered

→ Check producer repo has:
- Trigger logic in release workflow
- Organization PAT with correct permissions
- Correct workflow file name in search

### Container updates not detected

→ Verify:
- Container image is from `ghcr.io/morepet/containers/*`
- Container file is in supported location
- Renovate config includes correct manager

### Token expired

→ Fine-grained PATs expire after 90 days, create new one

## Best Practices

- **Pin action versions** - Let Renovate update them
- **Enable branch protection** - Require PR reviews and CI
- **Monitor Actions tab** - Check for failed workflows
- **Review security PRs promptly** - They auto-merge after CI
