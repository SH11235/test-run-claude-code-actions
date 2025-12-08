# test-run-claude-code-actions

A testing repository for Claude Code Actions with self-hosted GitHub Actions runner integration.

## Overview

This repository demonstrates how to use [Claude Code Actions](https://github.com/anthropics/claude-code-action) with a self-hosted GitHub Actions runner. It provides automated workflows that allow Claude AI to process GitHub issues, create pull requests, and interact with your repository through GitHub Actions.

## Workflows

### Active Workflows

#### claude-auto-scheduler.yml

**Purpose**: Automatically processes GitHub issues labeled with `claude` on a scheduled basis.

**Trigger**:
- Scheduled execution at 1:00 AM, 3:00 AM, and 5:00 AM JST (16:00, 18:00, 20:00 UTC)
- Manual execution via `workflow_dispatch`

**How it works**:
1. **Find Issues**: Searches for open issues with the `claude` label (excludes issues with `claude-error` label)
2. **Process Issues**: Processes up to 3 issues in parallel, with each issue:
   - Checked out from the repository
   - Passed to Claude Code Action with the issue title and body
   - Claude attempts to complete the task and create a pull request
3. **Update Status**:
   - **Success**: Removes `claude` label, adds `pr-created` label, and comments with PR link
   - **Failure**: Adds `claude-error` label and comments with error information

**How to use**:
1. Create or find an issue describing the task you want automated
2. Add the `claude` label to the issue
3. Wait for the next scheduled run (or manually trigger the workflow)
4. Claude will process the issue and attempt to create a pull request
5. Check the issue comments for status updates

**Features**:
- Processes up to 3 issues per run
- Continues processing other issues even if one fails (`fail-fast: false`)
- Automatically cleans up node_modules after execution to save disk space
- 30-minute timeout per issue

#### claude-code-self-hosted.yml

**Purpose**: Responds to `@claude` mentions in issues and comments in real-time.

**Trigger**:
- Issue comments containing `@claude`
- New issues containing `@claude` in the body

**How it works**:
1. Detects `@claude` mention in issue or comment
2. Runs Claude Code Action with the context of the issue/comment
3. Claude responds based on the instructions in the comment
4. Automatically cleans up node_modules after execution

**How to use**:
1. Create an issue or comment mentioning `@claude` followed by your request
2. Example: `@claude please add unit tests for the authentication module`
3. The workflow triggers automatically and Claude processes the request
4. Claude will respond or create a pull request as appropriate

**Features**:
- Real-time response to mentions
- Runs on self-hosted runner
- 30-minute timeout
- Full permissions for creating PRs and modifying issues

### Disabled Workflows

The following workflow files are currently disabled (with `.disabled` extension):

#### claude.yml.disabled

A general-purpose Claude Code workflow that responds to `@claude` mentions across various GitHub events (issues, PR comments, PR reviews). Originally configured for `ubuntu-latest` runners.

#### claude-code-review.yml.disabled

An automated code review workflow that would run on pull requests. Configured to provide feedback on code quality, potential bugs, performance, security concerns, and test coverage.

## Setup Instructions

### Prerequisites

- A GitHub repository
- A self-hosted GitHub Actions runner configured for your repository
- Claude Code OAuth token from Anthropic

### Self-Hosted Runner Setup

1. **Install the runner** on your machine:
   ```bash
   # Download the latest runner
   mkdir actions-runner && cd actions-runner
   curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
   tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz
   ```

2. **Configure the runner**:
   ```bash
   # Navigate to your repository Settings > Actions > Runners
   # Follow the instructions to generate a token
   ./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPO --token YOUR_TOKEN
   ```

3. **Run the runner**:
   ```bash
   # Run interactively
   ./run.sh

   # Or install as a service
   sudo ./svc.sh install
   sudo ./svc.sh start
   ```

### Required Secrets

Configure the following secret in your repository:

1. Go to **Settings > Secrets and variables > Actions**
2. Add a new repository secret:
   - **Name**: `CLAUDE_CODE_OAUTH_TOKEN`
   - **Value**: Your Claude Code OAuth token from Anthropic

To obtain a Claude Code OAuth token:
- Visit the [Anthropic Console](https://console.anthropic.com/)
- Navigate to the API Keys or OAuth section
- Generate a new OAuth token for GitHub Actions

### Allowed Tools Configuration

Both active workflows restrict Claude to specific tools for security:
- `Write`, `Edit`, `Read` - File operations
- `Glob`, `Grep` - Code search
- `Bash(git:*)` - Git commands
- `Bash(gh:*)` - GitHub CLI commands
- `Bash(ls:*)`, `Bash(pwd:*)`, `Bash(cd:*)` - Basic shell navigation
- `Bash(echo:*)`, `Bash(jq:*)` - Output and JSON processing

## Using the Auto-Scheduler Feature

The auto-scheduler is the primary way to automate tasks with Claude Code:

### Basic Usage

1. **Create an issue** describing your task:
   ```
   Title: Add user authentication
   Body: Please implement JWT-based authentication with login and logout endpoints.
   ```

2. **Add the `claude` label** to the issue

3. **Wait for processing**:
   - The workflow runs at scheduled times (1 AM, 3 AM, 5 AM JST)
   - Or manually trigger it from the Actions tab

4. **Review the results**:
   - If successful: PR is created and linked in the issue
   - If failed: Error information is added to the issue

### Label System

- **`claude`**: Marks an issue for processing by Claude
- **`pr-created`**: Added when Claude successfully creates a PR
- **`claude-error`**: Added when processing fails (issue can be retried)

### Best Practices

- Write clear, specific issue descriptions
- Include relevant context and requirements
- One task per issue for better tracking
- Review and test PRs created by Claude before merging
- Remove `claude-error` label and keep `claude` label to retry failed issues

## Permissions

The workflows require the following permissions:
- `contents: write` - Create branches and commits
- `pull-requests: write` - Create and update pull requests
- `issues: write` - Update issue labels and add comments
- `id-token: write` - Authenticate with Anthropic services
- `actions: read` - Read workflow run information

## Troubleshooting

### Runner Issues

- Ensure the self-hosted runner is online and connected
- Check runner logs: `journalctl -u actions.runner.*`
- Verify sufficient disk space (workflows clean up node_modules automatically)

### Workflow Failures

- Check the Actions tab for detailed error logs
- Review issue comments for Claude-specific error messages
- Ensure the `CLAUDE_CODE_OAUTH_TOKEN` secret is correctly configured
- Verify the runner has necessary permissions (git, gh CLI)

### Token Issues

- Ensure your OAuth token is valid and not expired
- Verify the token has the necessary scopes
- Check the token is correctly added as a repository secret

## Contributing

This is a testing repository. Feel free to create issues with the `claude` label to test the automation workflows.

## License

This is a testing/demonstration repository. Use at your own discretion.
