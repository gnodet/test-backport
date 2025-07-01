# Auto Port Workflow Setup Guide

This guide explains how to set up and use the automated backport/forward-port workflow for your repository.

## Overview

The workflow automatically creates backport and forward-port pull requests when specific labels are applied to PRs:

- **Backport**: Creates a PR to port changes from `master` to `maven-4.0.x` when labeled with `backport-to-4.0.x`
- **Forward-port**: Creates a PR to port changes from `maven-4.0.x` to `master` when labeled with `forward-port-to-master`

## Setup Instructions

### 1. Fix GitHub Actions Permissions Issue

The error "GitHub Actions is not permitted to create or approve pull requests" occurs because the default `GITHUB_TOKEN` has limited permissions. Here are two solutions:

#### Option A: Use Personal Access Token (Recommended)

1. **Create a Personal Access Token (PAT):**
   - Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Click "Generate new token (classic)"
   - Give it a descriptive name like "Auto Port Workflow"
   - Set expiration (recommend 1 year or no expiration for automation)
   - Select these scopes:
     - `repo` (Full control of private repositories)
     - `workflow` (Update GitHub Action workflows)

2. **Add the Token to Repository Secrets:**
   - Go to your repository → Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `AUTO_PORT_TOKEN`
   - Value: Paste the PAT you just created
   - Click "Add secret"

#### Option B: Try with Enhanced Permissions (Alternative)

If you prefer not to use a PAT, the workflow will fall back to using `GITHUB_TOKEN` with enhanced permissions. This may work in some repository configurations.

### 2. Repository Settings

Ensure your repository has the following branches:
- `master` (main development branch)
- `maven-4.0.x` (stable branch for backports)

### 3. Labels Setup

Create these labels in your repository (Settings → Labels):
- `backport-to-4.0.x` - Apply to PRs targeting `master` that should be backported
- `forward-port-to-master` - Apply to PRs targeting `maven-4.0.x` that should be forward-ported
- `auto-port` - Automatically applied to generated port PRs

## Usage

### Creating a Backport

1. Create a PR targeting the `master` branch
2. Add the label `backport-to-4.0.x` to the PR
3. The workflow will automatically:
   - Create a new branch `backport-{PR_NUMBER}-to-maven-4.0.x`
   - Cherry-pick commits from your PR
   - Create a new PR targeting `maven-4.0.x`
   - Mark the PR as draft if conflicts are detected

### Creating a Forward-port

1. Create a PR targeting the `maven-4.0.x` branch
2. Add the label `forward-port-to-master` to the PR
3. The workflow will automatically:
   - Create a new branch `backport-{PR_NUMBER}-to-master`
   - Cherry-pick commits from your PR
   - Create a new PR targeting `master`
   - Mark the PR as draft if conflicts are detected

## Handling Conflicts

If the workflow detects conflicts during cherry-picking:
- The generated PR will be marked as a draft
- The PR description will include a warning about conflicts
- You'll need to manually resolve conflicts in the generated branch
- Once resolved, convert the draft PR to ready for review

## Troubleshooting

### Common Issues

1. **"GitHub Actions is not permitted to create or approve pull requests"**
   - Follow the PAT setup instructions above

2. **Workflow doesn't trigger**
   - Ensure the labels are spelled correctly
   - Check that the PR targets the correct base branch
   - Verify the workflow file is in `.github/workflows/`

3. **Cherry-pick conflicts**
   - This is normal for complex changes
   - Manually resolve conflicts in the generated branch
   - The workflow will mark such PRs as drafts

### Debugging

Check the Actions tab in your repository to see workflow runs and any error messages.

## Workflow Features

- **Automatic branch cleanup**: Deletes existing port branches before creating new ones
- **Conflict detection**: Marks PRs as drafts when conflicts occur
- **Comprehensive PR descriptions**: Includes links to original PRs and conflict warnings
- **Label management**: Automatically applies `auto-port` label to generated PRs
- **Fork support**: Works with PRs from forked repositories

## Security Considerations

- The PAT has broad repository access - consider using a dedicated service account
- The workflow uses `pull_request_target` which runs in the context of the target repository
- All generated PRs should be reviewed before merging
