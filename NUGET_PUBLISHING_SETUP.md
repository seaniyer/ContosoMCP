# NuGet Trusted Publishing Setup Instructions

## What I've Done

1. **Incremented the package version** from 0.2.0-beta to 0.3.0-beta in:
   - `ContosoMCP/ContosoMCP.csproj`
   - `ContosoMCP/.mcp/server.json`

2. **Updated the GitHub Actions workflow** (`.github/workflows/publish-nuget.yml`) to:
   - Support multiple triggers: releases, version tags (v*), and manual dispatch
   - Use the `NuGet/login@v1` action to obtain a short-lived OIDC token
   - Use the temporary API key for publishing (valid for 1 hour)
   - Specify the correct project path

## Next Steps: Configure Trusted Publishing on NuGet.org

To complete the trusted publishing setup, you need to configure NuGet.org to trust your GitHub repository. Follow these steps:

### 1. Sign in to NuGet.org
   - Go to https://www.nuget.org
   - Sign in with your account

### 2. Configure Trusted Publishers
   - Click your username and choose **Trusted Publishing**
   - Click **Add a new trusted publishing policy**

### 3. Add GitHub Actions Configuration
   Fill in the following details (case-insensitive):
   - **Repository Owner**: `seaniyer`
   - **Repository**: `ContosoMCP`
   - **Workflow File**: `publish-nuget.yml` (just the filename, NOT the full path)
   - **Environment**: Leave blank (unless you're using GitHub environments like `environment: release`)

### 4. Choose Policy Owner
   - Select who owns this policy (you or an organization you belong to)
   - The policy will apply to all packages owned by the selected owner

### 5. Add GitHub Secret for NuGet Username
   - Go to your GitHub repository settings
   - Navigate to **Secrets and variables** ? **Actions**
   - Click **New repository secret**
   - Name: `NUGET_USER`
   - Value: Your NuGet.org **username** (profile name), NOT your email address
   - Click **Add secret**

## How It Works

1. Your GitHub Actions workflow runs
2. The `NuGet/login@v1` action requests an OIDC token from GitHub
3. The token is sent to nuget.org with your repository/workflow information
4. NuGet.org validates the token and issues a short-lived API key (valid for 1 hour)
5. The workflow uses that temporary key to publish your package
6. Each token can only be used once to obtain a single API key

## How to Publish

Once trusted publishing is configured on NuGet.org and the `NUGET_USER` secret is set, you can publish your package by:

### Option 1: Create a GitHub Release
```bash
# Create and push a tag
git tag v0.3.0-beta
git push origin v0.3.0-beta

# Then create a release on GitHub from that tag
```

### Option 2: Push a Version Tag
```bash
git tag v0.3.0-beta
git push origin v0.3.0-beta
```

### Option 3: Manual Trigger
- Go to your repository on GitHub
- Navigate to **Actions** ? **Publish NuGet Package**
- Click **Run workflow**
- Select the branch and click **Run workflow**

## Important Notes

- **No long-lived API key needed**: With trusted publishing, you don't need to store a `NUGET_API_KEY` secret in GitHub
- **Short-lived tokens**: The OIDC token and temporary API key are valid for 1 hour
- **One-time use**: Each OIDC token can only be used once to obtain a single temporary API key
- **First-time publish**: If this is the first time publishing this package, you may need to reserve the package ID on NuGet.org first
- **Private repos**: If you have a private GitHub repo, the policy starts as "temporarily active" for 7 days. After a successful publish, it becomes permanently active.

## Verification

After the workflow runs, you can verify the package was published at:
https://www.nuget.org/packages/ContosoMCPServer/

## Security Benefits

- No long-lived secrets to rotate or manage
- Reduced risk of credential leaks
- Cryptographically signed tokens that can't be tampered with
- Automatic validation by NuGet.org with GitHub

## References

- [Trusted Publishing Documentation](https://learn.microsoft.com/en-us/nuget/nuget-org/trusted-publishing)
- [GitHub Actions Setup](https://learn.microsoft.com/en-us/nuget/nuget-org/trusted-publishing#github-actions-setup)
- [OpenSSF Trusted Publishers Initiative](https://repos.openssf.org/trusted-publishers-for-all-package-repositories)
