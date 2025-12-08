# Claude Code Actions Testing with Self-Hosted Runner

This repository is a testing environment for running [Claude Code](https://docs.anthropic.com/en/docs/claude-code) actions on a self-hosted GitHub Actions runner. It demonstrates how to integrate Anthropic's Claude AI as an automated code assistant for handling GitHub Issues and pull requests.

## Overview

This repository contains GitHub Actions workflows that enable Claude Code to automatically process issues and interact with your repository through a self-hosted runner. Claude can read issue descriptions, implement requested changes, and create pull requests autonomously.

## Workflows

### 1. Claude Auto Scheduler (`claude-auto-scheduler.yml`)

**Purpose**: Automatically processes issues labeled with `claude` on a scheduled basis.

**Features**:
- Runs on a schedule (1:00 AM, 3:00 AM, and 5:00 AM JST)
- Can also be triggered manually via workflow_dispatch
- Finds up to 3 open issues with the `claude` label
- Processes each issue in parallel (with fail-fast disabled)
- Creates pull requests to address the issues
- Automatically updates issue labels based on success/failure:
  - On success: removes `claude` label, adds `pr-created` label
  - On failure: adds `claude-error` label, keeps `claude` label for retry
- Posts comments on issues with results and links to PRs or workflow runs
- Includes cleanup step to remove node_modules after execution

**Scheduled Times** (UTC):
- 16:00 UTC (1:00 AM JST)
- 18:00 UTC (3:00 AM JST)
- 20:00 UTC (5:00 AM JST)

**How to Use**:
1. Add the `claude` label to any open issue you want Claude to process
2. Wait for the next scheduled run, or manually trigger the workflow
3. Claude will read the issue, implement the requested changes, and create a PR
4. Check the issue for a comment with the PR link or error details

**Permissions Required**:
- `contents: write` - To create branches and commits
- `pull-requests: write` - To create pull requests
- `issues: write` - To update labels and add comments
- `id-token: write` - For Claude Code authentication
- `actions: read` - To read workflow information

### 2. Claude Code Self-Hosted (`claude-code-self-hosted.yml`)

**Purpose**: Responds to manual mentions of `@claude` in issue comments or issue bodies.

**Features**:
- Triggers when `@claude` is mentioned in:
  - New issue descriptions
  - Issue comments
- Runs on self-hosted runner
- 30-minute timeout
- Uses Claude to respond based on the context
- Includes cleanup step to remove node_modules after execution

**How to Use**:
1. Create a new issue and mention `@claude` in the body, or
2. Comment on an existing issue with `@claude` and your request
3. The workflow will trigger automatically
4. Claude will process your request and respond accordingly

**Permissions Required**:
- `contents: write` - To create branches and commits
- `pull-requests: write` - To create pull requests
- `issues: write` - To add comments
- `id-token: write` - For Claude Code authentication
- `actions: read` - To read workflow information

### Disabled Workflows

The following workflows are currently disabled (`.disabled` extension):

- **`claude.yml.disabled`**: A general-purpose Claude workflow that responds to `@claude` mentions across issues, PR comments, and PR reviews on ubuntu-latest runners
- **`claude-code-review.yml.disabled`**: Automated code review workflow that runs on pull requests using Claude to provide feedback on code quality, bugs, performance, and security

These workflows can be re-enabled by removing the `.disabled` extension from the filename.

## Setup Instructions

### Prerequisites

1. A GitHub repository
2. A self-hosted GitHub Actions runner configured and connected to your repository
3. A Claude Code OAuth token from Anthropic

### Self-Hosted Runner Setup

1. **Install the GitHub Actions runner** on your machine:
   ```bash
   # Create a directory for the runner
   mkdir actions-runner && cd actions-runner

   # Download the latest runner package
   curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

   # Extract the installer
   tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz
   ```

2. **Configure the runner**:
   ```bash
   # Navigate to your repository Settings > Actions > Runners > New self-hosted runner
   # Follow the configuration instructions provided by GitHub

   ./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPO --token YOUR_REGISTRATION_TOKEN
   ```

3. **Run the runner**:
   ```bash
   # Run interactively
   ./run.sh

   # Or install as a service (Linux)
   sudo ./svc.sh install
   sudo ./svc.sh start
   ```

### Required Secrets

Configure the following secret in your repository settings (`Settings > Secrets and variables > Actions`):

- **`CLAUDE_CODE_OAUTH_TOKEN`**: Your Claude Code OAuth token
  - Obtain this token from [Anthropic's Claude platform](https://console.anthropic.com/)
  - This token allows the GitHub Action to authenticate with Claude Code services

The `GITHUB_TOKEN` is automatically provided by GitHub Actions and does not need to be configured.

## Using the Auto-Scheduler Feature

The auto-scheduler feature allows you to queue issues for Claude to process automatically.

### How to Use:

1. **Create or identify an issue** that you want Claude to handle

2. **Add the `claude` label** to the issue:
   - Through the GitHub UI: Click "Labels" on the right sidebar and select `claude`
   - Via GitHub CLI: `gh issue edit ISSUE_NUMBER --add-label "claude"`

3. **Wait for automatic processing**:
   - The scheduler runs three times per night (1 AM, 3 AM, 5 AM JST)
   - Or manually trigger the workflow from the Actions tab

4. **Monitor the results**:
   - Claude will post a comment on the issue when processing is complete
   - On success: A PR link will be provided and the issue will be labeled `pr-created`
   - On failure: An error message will be posted and the issue will be labeled `claude-error`

5. **Review the pull request**:
   - Check the PR created by Claude
   - Review the changes and merge if acceptable
   - Provide feedback if changes are needed

### Labels Used:

- `claude` - Marks an issue for automatic processing (removed on success)
- `pr-created` - Added when a PR is successfully created
- `claude-error` - Added when processing fails (claude label kept for retry)

## Allowed Tools Configuration

Both active workflows configure Claude with specific allowed tools for security:

```yaml
claude_args: '--allowed-tools "Write" "Edit" "Read" "Glob" "Grep" "Bash(git:*)" "Bash(gh:*)" "Bash(ls:*)" "Bash(pwd:*)" "Bash(cd:*)" "Bash(echo:*)" "Bash(jq:*)"'
```

This restricts Claude to:
- File operations: Write, Edit, Read, Glob, Grep
- Git commands: All git operations
- GitHub CLI: All gh commands
- Basic shell: ls, pwd, cd, echo, jq

## Troubleshooting

- **Runner not picking up jobs**: Ensure your self-hosted runner is online and properly configured
- **Authentication errors**: Verify your `CLAUDE_CODE_OAUTH_TOKEN` is correctly set in repository secrets
- **Permission errors**: Check that your workflows have the required permissions in the job configuration
- **Timeout issues**: Increase the `timeout-minutes` value if tasks take longer than 30 minutes
- **Node modules disk space**: The workflows include cleanup steps to remove node_modules after each run

## Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code Action GitHub Repository](https://github.com/anthropics/claude-code-action)
- [GitHub Actions Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners)
- [Anthropic Console](https://console.anthropic.com/)

## License

This is a testing repository. Please refer to [Anthropic's terms of service](https://www.anthropic.com/legal/consumer-terms) for Claude Code usage.
