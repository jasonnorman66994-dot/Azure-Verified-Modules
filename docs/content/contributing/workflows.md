---
title: GitHub Actions Workflows
description: Documentation for GitHub Actions workflows used in the Azure Verified Modules repository
---

## Summary

This page documents the GitHub Actions workflows used in the Azure Verified Modules (AVM) repository. These workflows automate various aspects of repository maintenance, documentation, quality assurance, and module publishing.

## Workflow Categories

### Documentation & Website

#### Deploy Hugo site to Pages

**File:** `hugo-site-build.yml`

**Purpose:** Builds and deploys the AVM documentation website to GitHub Pages using Hugo.

**Triggers:**
- Push to `main` branch with changes in `docs/**`
- Scheduled daily at 1 AM UTC (cron: `0 1 * * *`)
- Manual workflow dispatch

**Key Features:**
- Uses Hugo version 0.136.5 with extended features
- Installs Dart Sass for styling
- Builds with minification and garbage collection
- Deploys to GitHub Pages environment

#### Hugo Build PR Check

**File:** `hugo-build-pr-check.yml`

**Purpose:** Validates that documentation changes in pull requests build successfully without errors.

**Triggers:**
- Pull requests with changes in `docs/**` or the workflow file itself
- Manual workflow dispatch

**Key Features:**
- Same build process as production deployment
- Ensures documentation changes don't break the site
- No actual deployment, build validation only

#### Deploy static content to Pages

**File:** `static.yml`

**Purpose:** Deploys static content to GitHub Pages (alternative to Hugo deployment).

**Triggers:**
- Push to `main` branch
- Manual workflow dispatch

**Key Features:**
- Simple static content deployment
- Uploads entire repository as artifact
- Single deployment job

### Code Quality & Review

#### Code Review - Linting & Link Checks

**File:** `code-review.yml`

**Purpose:** Performs automated code quality checks and validates markdown links.

**Triggers:**
- All pull requests
- Manual workflow dispatch

**Jobs:**
1. **Lint code base**: Uses GitHub Super Linter to validate:
   - JSON files
   - Markdown files
   - PowerShell scripts
   - YAML files
   - Bash scripts
   - EditorConfig compliance

2. **Markdown Link Check**: Validates all links in markdown files using custom configuration

**Key Features:**
- Only validates changed files (not entire codebase)
- Excludes specific paths (theme files, telemetry notices)
- Provides detailed feedback on PR checks

### Platform Automation

#### Update API Specs file

**File:** `platform.apiSpecs.yml`

**Purpose:** Automatically updates the Azure API Specs list used for module development.

**Triggers:**
- Scheduled weekly on Mondays at 10 AM UTC (cron: `0 10 * * 1`)
- Manual workflow dispatch

**Key Features:**
- Fetches latest API specs from Azure
- Includes preview and external sources
- Creates automated PR with updates
- Stores data in `docs/static/governance/apiSpecsList.json`

**Requirements:**
- Azure OIDC authentication
- GitHub App token for PR creation

#### Update module registry tables

**File:** `platform.updateModuleRegistryTables.yml`

**Purpose:** Updates module status badges and feature tables in documentation.

**Triggers:**
- Scheduled daily at 1 AM UTC (cron: `0 1 * * *`)
- Manual workflow dispatch

**Key Features:**
- Checks out Bicep registry modules
- Updates module status badge table
- Updates module features CSV
- Creates automated PR with changes
- Updates files:
  - `docs/static/includes/module-features/bicepBadges.md`
  - `docs/static/includes/module-features/bicepFeatures.csv`

**Requirements:**
- GitHub App token
- Access to Azure/bicep-registry-modules repository

#### Create AzAdvertizer diff issue

**File:** `platform.new-AzAdvertizer-diff-issue.yml`

**Purpose:** Monitors changes in AzAdvertizer data (PSRule, APRL, and Advisor) and creates issues when differences are detected.

**Triggers:**
- Scheduled weekly on Sundays at 3 AM UTC (cron: `0 3 * * 0`)
- Manual workflow dispatch with optional "what-if" mode

**Key Features:**
- Compares current AzAdvertizer data with previous run
- Creates GitHub issues for detected changes
- Uploads artifact with AzAdvertizer data CSV
- Supports simulation mode for testing

### Repository Governance

#### Sync AVM Labels from CSV to GitHub Labels

**File:** `csv-label-sync.yml`

**Purpose:** Synchronizes repository labels from a CSV file to ensure consistent labeling across the repository.

**Triggers:**
- Push to `main` with changes to `docs/static/governance/avm-standard-github-labels.csv`
- Manual workflow dispatch

**Key Features:**
- Waits for Hugo website deployment (120 seconds)
- Runs PowerShell script to sync labels
- Uses standard AVM label definitions

**Requirements:**
- Write permissions for contents, issues, and pull requests

#### GitHub Team - Check Existence

**File:** `github-teams-check-existence.yml`

**Purpose:** Validates that GitHub teams referenced in module configurations exist and have correct permissions.

**Triggers:**
- Scheduled weekdays at 3 PM UTC (cron: `0 15 * * 1-5`)
- Manual workflow dispatch

**Teams Validated:**
- Bicep Resource Owners
- Bicep Resource Contributors
- Bicep Pattern teams
- Terraform Resource teams
- Terraform Pattern teams

**Key Features:**
- Validates team existence
- Checks team permissions
- Creates issues for missing teams (main branch only)
- Validates parent configuration for Bicep modules

**Requirements:**
- GitHub App token with team access
- Azure PowerShell modules

#### Terraform repo governance

**File:** `terraform-repo-governance.yml`

**Purpose:** Ensures Terraform AVM repositories comply with governance requirements.

**Triggers:**
- Scheduled weekly on Sundays at 12:43 AM UTC (cron: `43 0 * * 0`)
- Manual workflow dispatch

**Key Features:**
- Discovers all Terraform AVM repositories using GitHub GraphQL API
- Validates MIT License presence (SNFR10 requirement)
- Creates issues for non-compliant repositories
- Uses matrix strategy for parallel repository checking

### Package Publishing

#### Upload Python Package

**File:** `python-publish.yml`

**Purpose:** Publishes Python packages to PyPI when releases are created.

**Triggers:**
- GitHub release published

**Jobs:**
1. **release-build**: Builds Python distributions
2. **pypi-publish**: Publishes to PyPI using trusted publishing

**Key Features:**
- Uses Python 3.x
- Trusted publishing with OIDC
- Artifact-based build and publish workflow
- Dedicated PyPI environment

**Requirements:**
- PyPI trusted publishing configured
- PyPI environment with deployment protections

## Common Workflow Patterns

### Authentication Methods

Workflows use different authentication methods depending on their needs:

1. **GitHub App Tokens**: For operations requiring elevated permissions
   - Used in: API Specs, Module Registry, Team Linter, AzAdvertizer workflows
   - Provides granular permissions and audit trail

2. **GITHUB_TOKEN**: For standard repository operations
   - Used in: Code review, Label sync, Hugo builds
   - Automatically provided by GitHub Actions

3. **Azure OIDC**: For Azure resource access
   - Used in: API Specs workflow
   - Enables passwordless authentication to Azure

### Pull Request Creation Pattern

Several workflows automatically create pull requests:

1. Check if branch exists
2. Create or checkout branch
3. Make changes
4. Check for git changes
5. Commit and push if changes detected
6. Create PR if it doesn't exist

**Workflows using this pattern:**
- Update API Specs file
- Update module registry tables

### Scheduled Automation

| Workflow | Schedule | Frequency |
|----------|----------|-----------|
| Hugo site build | `0 1 * * *` | Daily at 1 AM |
| Module registry tables | `0 1 * * *` | Daily at 1 AM |
| API Specs update | `0 10 * * 1` | Weekly Monday at 10 AM |
| AzAdvertizer diff | `0 3 * * 0` | Weekly Sunday at 3 AM |
| Team existence check | `0 15 * * 1-5` | Weekdays at 3 PM |
| Terraform governance | `43 0 * * 0` | Weekly Sunday at 12:43 AM |

## Troubleshooting

### Common Issues

**Hugo Build Failures:**
- Check Hugo version compatibility (currently 0.136.5)
- Verify all markdown syntax is valid
- Ensure all referenced files and images exist

**Label Sync Issues:**
- Verify CSV file format in `docs/static/governance/avm-standard-github-labels.csv`
- Check that GITHUB_TOKEN has required permissions

**Team Linter Failures:**
- Ensure GitHub teams exist in the organization
- Verify team names match module configuration
- Check GitHub App has appropriate organization permissions

**Authentication Errors:**
- Verify secrets are configured in repository settings
- Check that GitHub App has required permissions
- Ensure Azure OIDC configuration is current

## Best Practices

1. **Always test workflow changes**: Use `workflow_dispatch` to manually test changes
2. **Monitor scheduled runs**: Check workflow run history for failures
3. **Review automated PRs promptly**: Don't let automated PRs accumulate
4. **Keep secrets updated**: Rotate credentials regularly
5. **Use branch protection**: Require workflow success before merging

## Contributing to Workflows

When modifying workflows:

1. Test changes in a fork or feature branch
2. Use `workflow_dispatch` for testing
3. Document new environment variables or secrets needed
4. Update this documentation with workflow changes
5. Follow GitHub Actions best practices
6. Minimize workflow runs to conserve resources

## Related Documentation

- [Contributing Process]({{% siteparam base %}}/contributing/process/)
- [Bicep Contribution Guide]({{% siteparam base %}}/contributing/bicep/)
- [Terraform Contribution Guide]({{% siteparam base %}}/contributing/terraform/)
- [Enable or Disable Workflows]({{% siteparam base %}}/contributing/bicep/bicep-contribution-flow/enable-or-disable-workflows/)
