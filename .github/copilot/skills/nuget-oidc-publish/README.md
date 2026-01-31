# NuGet OIDC Trusted Publishing Skill

This skill helps you create or migrate GitHub Actions workflows to publish NuGet packages to NuGet.org using **OIDC (OpenID Connect) Trusted Publishing** - the modern, secure alternative to long-lived API keys.

## What This Skill Does

1. **Discovers** existing CI/CD pipelines in your repository
2. **Analyzes** current NuGet publishing configuration
3. **Reports findings** to you before making any changes
4. **Creates or migrates** workflows to use OIDC authentication
5. **Guides you** through setting up Trusted Publishing on NuGet.org

## When to Use This Skill

- "Create a GitHub Actions workflow to publish my NuGet package"
- "Migrate my NuGet publishing to use OIDC instead of API keys"
- "Set up secure NuGet publishing without secrets"
- "Configure trusted publishing for my .NET library"

## How It Works

1. GitHub Actions workflow runs with `id-token: write` permission
2. `NuGet/login@v1` action exchanges the GitHub OIDC token for a **short-lived NuGet API key**
3. This temporary API key is used with `dotnet nuget push`
4. The key expires automatically after the workflow completes

## Key Benefits vs Long-Lived API Keys

| Feature | Legacy API Key | OIDC Trusted Publishing |
|---------|---------------|------------------------|
| Secrets | Long-lived, can leak | Short-lived, auto-generated |
| Token Lifetime | Never expires | Minutes |
| Rotation | Manual rotation needed | Automatic per-run |
| Scope | All packages | Per repository/workflow |
| Storage | Repository secrets | No secret storage needed |

## Files in This Skill

- `SKILL.md` - Main skill instructions
- `templates/` - Reference workflow templates
- `README.md` - This file

## Quick Start

Ask your AI assistant:
> "Help me set up NuGet OIDC publishing for my package"

The assistant will guide you through the entire process.
