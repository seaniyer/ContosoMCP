````skill
---
name: nuget-oidc-publish
description: Creates or migrates GitHub Actions workflows to publish NuGet packages to NuGet.org using OIDC Trusted Publishing. Use when user asks to create, set up, configure, or migrate NuGet publishing pipelines, CI/CD for NuGet, or secure package publishing without API keys.
---

# NuGet OIDC Trusted Publishing Skill

## Overview

This skill helps users create or migrate GitHub Actions workflows to publish NuGet packages to NuGet.org using **OIDC (OpenID Connect) Trusted Publishing** instead of long-lived API keys.

## Critical Instructions

- **NEVER** use long-lived NuGet API keys. Always use OIDC-based trusted publishing.
- **Always inform the user** about what was found and what changes are planned before making modifications.
- **Get confirmation** before migrating existing pipelines.
- **Guide the user** through setting up Trusted Publishing on NuGet.org after creating/updating the workflow.

## Why OIDC Trusted Publishing?

| Feature | API Key (Legacy) | OIDC Trusted Publishing |
|---------|-----------------|------------------------|
| Security | Long-lived secret that can be leaked | Short-lived tokens, no stored secrets |
| Rotation | Manual rotation required | Automatic, no rotation needed |
| Scope | Broad access to all packages | Scoped to specific repository/workflow |
| Audit | Limited traceability | Full audit trail with GitHub Actions |
| Setup | Store secret in repository | One-time configuration on NuGet.org |

## Workflow Steps

### Step 1: Discovery & Analysis

Before making any changes, thoroughly analyze the repository:

```markdown
Discovery Checklist:
- [ ] Search for existing GitHub Actions workflows (.github/workflows/*.yml)
- [ ] Search for existing Azure Pipelines (azure-pipelines.yml, .azure-pipelines/)
- [ ] Search for NuGet API key usage (NUGET_API_KEY, nuget push, dotnet nuget push)
- [ ] Identify .csproj files with package configuration
- [ ] Check for existing NuGet.config files
```

**Search patterns to use:**
- `.github/workflows/*.yml` - GitHub Actions workflows
- `azure-pipelines.yml`, `.azure-pipelines/*.yml` - Azure Pipelines
- Look for `nuget push`, `dotnet nuget push`, `NUGET_API_KEY`, `NUGET_TOKEN` in workflow files

### Step 2: Report Findings to User

**Always present findings clearly before proceeding:**

```markdown
## Pipeline Analysis Results

### Existing Workflows Found:
- [List each workflow file with its purpose]
- [Highlight any that publish to NuGet]

### Current NuGet Publishing Method:
- [API Key / None / Other]

### Recommended Actions:
- [Create new / Migrate existing / Both]
```

**Ask for confirmation:**
- If existing NuGet publishing workflows are found, ask: "I found [X] existing workflow(s) that publish to NuGet. Would you like me to migrate these to use OIDC, or create a new workflow?"
- Let user choose which workflows to migrate if multiple exist.

### Step 3: Create/Update GitHub Actions Workflow

Use this **reference workflow template** for OIDC-based NuGet publishing:

```yaml
name: Publish NuGet Package

on:
  push:
    tags:
      - 'v*'  # Trigger on version tags like v1.0.0, v0.3.0-beta
  release:
    types: [published]
  workflow_dispatch:  # Allow manual trigger

jobs:
  publish:
    runs-on: ubuntu-latest

    # IMPORTANT: Required for OIDC token authentication with NuGet.org
    permissions:
      id-token: write  # Required for trusted publishing (GitHub OIDC token)
      contents: read

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --configuration Release --no-restore

      - name: Run tests
        run: dotnet test --configuration Release --no-build --verbosity normal

      - name: Pack
        run: dotnet pack --configuration Release --no-build --output ./nupkg

      # OIDC: Exchange GitHub token for short-lived NuGet API key
      - name: NuGet login (OIDC → temp API key)
        uses: NuGet/login@v1
        id: nuget-login
        with:
          user: YOUR_NUGET_USERNAME  # Your nuget.org username (profile name), NOT email

      - name: Publish to NuGet.org (Trusted Publishing)
        run: |
          dotnet nuget push ./nupkg/*.nupkg \
            --api-key ${{ steps.nuget-login.outputs.NUGET_API_KEY }} \
            --source https://api.nuget.org/v3/index.json \
            --skip-duplicate
```

### Key Configuration Points

**1. Permissions Block (CRITICAL):**
```yaml
permissions:
  id-token: write  # REQUIRED for OIDC authentication
  contents: read   # REQUIRED for checkout
```
This enables the workflow to request an OIDC token from GitHub.

**2. NuGet Login Action (CRITICAL):**
```yaml
- name: NuGet login (OIDC → temp API key)
  uses: NuGet/login@v1
  id: nuget-login
  with:
    user: YOUR_NUGET_USERNAME  # nuget.org profile name, NOT email
```
This action exchanges the GitHub OIDC token for a **short-lived NuGet API key**.

**3. Using the Temporary API Key:**
```yaml
- name: Publish to NuGet.org
  run: |
    dotnet nuget push ./nupkg/*.nupkg \
      --api-key ${{ steps.nuget-login.outputs.NUGET_API_KEY }} \
      --source https://api.nuget.org/v3/index.json \
      --skip-duplicate
```
The API key from `NuGet/login@v1` is short-lived and scoped - much safer than long-lived keys.

**4. Trigger Options:**
- `push: tags: ['v*']` - Publish on version tags (recommended)
- `release: [published]` - Publish on GitHub release
- `workflow_dispatch` - Manual trigger

### Step 4: Guide User to Configure NuGet.org Trusted Publishing

After creating/updating the workflow, provide these instructions:

```markdown
## Next Steps: Configure Trusted Publishing on NuGet.org

You need to configure Trusted Publishing on NuGet.org to allow this workflow to publish packages.

### For New Packages:
1. Go to https://www.nuget.org/packages/manage/upload
2. Start uploading your first package version
3. During upload, you'll see "Add trusted publisher" option
4. Configure the GitHub Actions trusted publisher

### For Existing Packages:
1. Go to https://www.nuget.org/account/Packages
2. Click on your package name
3. Go to "Trusted publishers" tab (or "Manage package" > "Trusted publishers")
4. Click "Add trusted publisher"
5. Select "GitHub Actions"

### Required Configuration Values:
| Field | Value |
|-------|-------|
| **Repository owner** | `{owner}` |
| **Repository name** | `{repo}` |
| **Workflow filename** | `{workflow-filename}.yml` |
| **Environment** | (leave empty unless using GitHub Environments) |

### Workflow Configuration:
Update the `NuGet/login@v1` step with your NuGet.org username:
```yaml
- name: NuGet login (OIDC → temp API key)
  uses: NuGet/login@v1
  id: nuget-login
  with:
    user: {nuget-username}  # Your nuget.org profile name (NOT email)
```

### Verification:
After configuration, run the workflow manually to test:
1. Go to Actions tab in your repository
2. Select the publish workflow
3. Click "Run workflow"
4. Monitor the run for successful NuGet push
```

**IMPORTANT:** Replace placeholders with actual values from the user's repository.

## Migration Scenarios

### Migrate from API Key-based GitHub Actions

**Before (Legacy Long-Lived API Key):**
```yaml
- name: Publish to NuGet
  run: dotnet nuget push **/*.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json
```

**After (OIDC with Short-Lived API Key):**
```yaml
jobs:
  publish:
    runs-on: ubuntu-latest
    
    permissions:
      id-token: write  # Required for OIDC
      contents: read

    steps:
      # ... build steps ...
      
      - name: NuGet login (OIDC → temp API key)
        uses: NuGet/login@v1
        id: nuget-login
        with:
          user: YOUR_NUGET_USERNAME

      - name: Publish to NuGet
        run: |
          dotnet nuget push ./nupkg/*.nupkg \
            --api-key ${{ steps.nuget-login.outputs.NUGET_API_KEY }} \
            --source https://api.nuget.org/v3/index.json \
            --skip-duplicate
```

**Changes Required:**
1. Add `permissions` block with `id-token: write` at job level
2. Add `NuGet/login@v1` step before the push step
3. Replace `${{ secrets.NUGET_API_KEY }}` with `${{ steps.nuget-login.outputs.NUGET_API_KEY }}`
4. Add your NuGet.org username to the login step
5. (Optional) Delete the `NUGET_API_KEY` secret from repository settings after verifying OIDC works

### Migrate from Azure Pipelines

If user has Azure Pipelines publishing to NuGet, offer to:
1. Create equivalent GitHub Actions workflow
2. Keep Azure Pipelines and help configure it for OIDC (if supported)
3. Explain the options and let user decide

## Troubleshooting Guide

### Common Issues:

**Error: "The request was blocked because it's from a workflow not trusted by the package owners"**
- Cause: Trusted publisher not configured on NuGet.org
- Solution: Complete Step 4 to configure trusted publishing

**Error: "403 Forbidden" or "Authentication required"**
- Cause: Missing `id-token: write` permission
- Solution: Add permissions block to workflow

**Error: "No packages were found with the specified pattern"**
- Cause: Pack step not producing nupkg files
- Solution: Check `dotnet pack` output directory matches push pattern

### Verification Commands:

```bash
# Test pack locally
dotnet pack --configuration Release --output ./nupkg

# List generated packages
ls ./nupkg/*.nupkg
```

## Security Best Practices

1. **Use specific triggers**: Prefer `release` or tag-based triggers over every push
2. **Use environments**: Consider GitHub Environments for additional approval gates
3. **Minimal permissions**: Only grant `id-token: write` and `contents: read`
4. **Branch protection**: Ensure main branch requires PR reviews
5. **After migration**: Remove deprecated `NUGET_API_KEY` secret from repository

## References

- [NuGet Trusted Publishing Documentation](https://learn.microsoft.com/en-us/nuget/nuget-org/trusted-publishing)
- [GitHub Actions OIDC Documentation](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [dotnet nuget push Reference](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-nuget-push)

````
