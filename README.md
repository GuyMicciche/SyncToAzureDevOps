# SyncToAzureDevOps

[![GitHub release](https://img.shields.io/github/v/release/GuyMicciche/SyncToAzureDevOps)](https://github.com/GuyMicciche/SyncToAzureDevOps/releases/latest)
[![Tests](https://github.com/GuyMicciche/SyncToAzureDevOps/workflows/Tests/badge.svg)](https://github.com/GuyMicciche/SyncToAzureDevOps/actions/workflows/ci.yml)

Bidirectional sync between GitHub and Azure DevOps — mirror a single repo or your entire organization, in either direction, on any trigger. Creates projects and repos automatically.

---

## Quick start

### GitHub to Azure DevOps (default)

```yaml
- name: Sync to Azure DevOps
  uses: GuyMicciche/SyncToAzureDevOps@v2.0.0
  with:
    organization: "dev.azure.com/yourorganization"
    pat: ${{ secrets.AZURE_DEVOPS_PAT }}
```

### Azure DevOps to GitHub (business continuity / DR backup)

```yaml
- name: Mirror Azure DevOps to GitHub
  uses: GuyMicciche/SyncToAzureDevOps@v2.0.0
  with:
    organization: "dev.azure.com/yourorganization"
    pat: ${{ secrets.AZURE_DEVOPS_PAT }}
    direction: "azure-to-github"
    sync-mode: "org-wide"
    github-token: ${{ secrets.GH_BACKUP_TOKEN }}
    github-org: "your-backup-github-org"
```

---

## Features

- **Auto-creates** Azure DevOps project and repository if missing
- **Auto-creates** GitHub repository if missing
- Syncs **all branches, tags, and full commit history**
- Bidirectional: GitHub to Azure or Azure to GitHub
- Single-repo or org-wide mirror mode
- **Zero config** for single-repo use — GitHub repo name is used automatically
- Works on **any trigger**: push, schedule, or manual dispatch

---

## Inputs

| Input | Required | Default | Description |
| :-- | :--: | :-- | :-- |
| `organization` | Yes | — | Azure DevOps org URL without `https://` (e.g. `dev.azure.com/myorg`) |
| `pat` | Yes | — | Azure DevOps PAT (see permissions below) |
| `direction` | No | `github-to-azure` | `github-to-azure` or `azure-to-github` |
| `sync-mode` | No | `single` | `single` (current repo only) or `org-wide` (all projects and repos) |
| `github-token` | No* | — | GitHub token with repo write access. Required when `direction` is `azure-to-github` |
| `github-org` | No | repo owner | Target GitHub org or user for `azure-to-github` mode |

---

## Requirements

### Azure DevOps PAT scopes

| Area | Permissions |
| :-- | :-- |
| Build | Read & execute |
| Code | Read, write & manage |
| Project and Team | Read, write & manage |

### GitHub token scopes (azure-to-github only)

| Scope | Why |
| :-- | :-- |
| `repo` | Create and push to private repositories |

---

## Setup

### 1. Create Azure DevOps PAT

1. Go to `dev.azure.com` and open **User settings > Personal access tokens > New Token**
2. Name it something like `GitHub Sync Action`
3. Set the organization and select the scopes from the table above
4. Copy the token immediately — it is only shown once

### 2. Create GitHub Personal Access Token (azure-to-github only)

1. Go to **github.com → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**
2. Name it `Azure DevOps Backup`
3. Check the `repo` scope
4. Click **Generate token**
5. Copy the token immediately — it is only shown once

### 3. Add secrets to GitHub

For GitHub to Azure sync:

```
Repo > Settings > Secrets and variables > Actions > New secret
Name:  AZURE_DEVOPS_PAT
Value: [Azure DevOps PAT (GitHub Sync Action value from Step 1)]
```

For Azure to GitHub sync, add a second secret:

```
Name:  GH_BACKUP_TOKEN
Value: [GitHub personal access token (Azure DevOps Backup value from Step 2)]
```

### 4. Add a workflow

See the examples below for common configurations.

---

## Example workflows

### On every push to main (GitHub to Azure)

```yaml
name: Sync to Azure DevOps

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: GuyMicciche/SyncToAzureDevOps@v2.0.0
        with:
          organization: "dev.azure.com/yourorganization"
          pat: ${{ secrets.AZURE_DEVOPS_PAT }}
```

### Nightly business continuity backup (Azure to GitHub, org-wide)

Mirrors every project and repo in your Azure DevOps organization to a GitHub org on a schedule. Designed for disaster recovery — if Azure becomes unavailable, your full history is preserved in GitHub.

```yaml
name: Azure DevOps Business Continuity Backup

on:
  schedule:
    - cron: "0 2 * * *"   # nightly at 02:00 UTC
  workflow_dispatch:

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: GuyMicciche/SyncToAzureDevOps@v2.0.0
        with:
          organization: "dev.azure.com/yourorganization"
          pat: ${{ secrets.AZURE_DEVOPS_PAT }}
          direction: "azure-to-github"
          sync-mode: "org-wide"
          github-token: ${{ secrets.GH_BACKUP_TOKEN }}
          github-org: "your-backup-github-org"
```

Repos are named `ProjectName--RepoName` in GitHub to avoid collisions when multiple Azure projects contain repos with the same name. Empty repos (Azure creates one per project by default) are detected and skipped automatically.

### CI/CD mirror (push and PR)

```yaml
name: CI/CD with Azure DevOps Mirror

on:
  push:
    branches: [ main, develop ]
  pull_request:
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: GuyMicciche/SyncToAzureDevOps@v2.0.0
        with:
          organization: "dev.azure.com/mycompany"
          pat: ${{ secrets.AZURE_DEVOPS_PAT }}
```

---

## Release notes

### v2.0.0

- New `direction` input: `github-to-azure` (default, preserves existing behavior) or `azure-to-github`
- New `sync-mode` input: `single` (default) or `org-wide` to mirror an entire Azure DevOps organization
- New `github-token` and `github-org` inputs for Azure-to-GitHub sync
- Org-wide mode iterates all projects and repos automatically, skipping empty repos
- Collision-safe naming in org-wide mode (`ProjectName--RepoName`)

### v1.0.0

- Auto-creates Azure DevOps **project** if missing
- Auto-creates **repository** if missing
- Syncs all branches, tags, and full commit history via `--all` push
- Zero-config single-repo mode using the GitHub repo name

---

## Links

- [Releases](https://github.com/GuyMicciche/SyncToAzureDevOps/releases)
- [Azure DevOps PAT guide](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)

---

⭐ **Star this repo** if it helps your workflow.
🔔 **Watch** for updates.
