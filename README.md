# SyncToAzureDevOps
[![GitHub release](https://img.shields.io/github/v/release/GuyMicciche/SyncToAzureDevOps)](https://github.com/GuyMicciche/SyncToAzureDevOps/releases/latest)
[![Tests](https://github.com/GuyMicciche/SyncToAzureDevOps/workflows/Tests/badge.svg)](https://github.com/GuyMicciche/SyncToAzureDevOps/actions/workflows/ci.yml)

**Automatically syncs your entire GitHub repository (branches + tags + history) to Azure DevOps** with one line in your workflow. Creates projects/repos automatically.

## 🚀 Quick Start

Add to your GitHub workflow:

```yaml
- name: Sync to Azure DevOps
  uses: GuyMicciche/SyncToAzureDevOps@v1.0.0
  with:
    organization: "dev.azure.com/yourorganization"
    pat: ${{ secrets.AZURE_DEVOPS_PAT }}
```


## ✨ Features

- ✅ **Auto-creates** Azure DevOps project if missing
- ✅ **Auto-creates** repository if missing
- ✅ Syncs **all branches + tags + full history**
- ✅ **Zero config** - uses GitHub repo name for project/repo
- ✅ Works in **any workflow** (push, schedule, manual)


## 📋 Requirements

1. **Azure DevOps PAT** with these scopes:


| Area | Permissions |
| :-- | :-- |
| Build | Read \& execute |
| Code | Read, write \& manage |
| Project and Team | Read, write \& manage |

2. **GitHub secret**: `AZURE_DEVOPS_PAT`

## 🛠 Complete Setup (5 minutes)

### 1. Create Azure DevOps PAT

```
1. dev.azure.com → User settings → Personal access tokens → New Token
2. Name: "GitHub Sync Action"
3. Organization: Your org
4. Scopes: Custom → Check Build/Code/Project permissions above
5. Copy token → Save immediately
```


### 2. Add GitHub Secret

```
Repo → Settings → Secrets and variables → Actions → New secret
Name: AZURE_DEVOPS_PAT
Value: [your PAT token]
```


### 3. Add to Workflow

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
      
      - name: Sync to Azure DevOps
        uses: GuyMicciche/SyncToAzureDevOps@v1.0.0
        with:
          organization: "dev.azure.com/yourorganization"
          pat: ${{ secrets.AZURE_DEVOPS_PAT }}
```


## 📱 Example Workflow

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
      - uses: GuyMicciche/SyncToAzureDevOps@v1.0.0
        with:
          organization: "dev.azure.com/mycompany"
          pat: ${{ secrets.AZURE_DEVOPS_PAT }}
```


## 🎯 Release Notes

### v1.0.0 – Initial Release

- Automatically creates Azure DevOps **project** if missing
- Automatically creates **repository** if missing
- Syncs entire GitHub repo including **branches/tags/history**
- Supports full history via `--all` push
- Zero-config: GitHub repo name used as default project/repo name


## 🔗 Links

- [Releases](https://github.com/GuyMicciche/SyncToAzureDevOps/releases)
- [Azure DevOps PAT Guide]([https://dev.azure.com](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)

---

⭐ **Star this repo** if it helps your workflow!
🔔 **Watch** for updates

---
