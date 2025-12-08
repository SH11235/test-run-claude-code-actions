# Claude Code Actions Testing Repository

This repository is designed for testing and demonstrating [Claude Code](https://claude.ai/claude-code) integration with GitHub Actions using a self-hosted runner. It provides automated workflows that allow Claude to process GitHub issues and create pull requests automatically.

## Overview

This project showcases how to set up Claude Code to automatically handle development tasks through GitHub Actions workflows. The primary use case is automated issue processing where Claude reads issue descriptions, implements the requested changes, and creates pull requests.

## Workflows

### Active Workflows

#### 1. `claude-auto-scheduler.yml` - Automated Issue Processing

This workflow automatically processes issues labeled with `claude` on a scheduled basis.

**Features:**
- Runs on a schedule (1 AM, 3 AM, and 5 AM Japan Time)
- Can also be triggered manually via workflow dispatch
- Processes up to 3 open issues with the `claude` label per run
- Creates pull requests for each issue
- Automatically manages issue labels based on processing results

**How it works:**
1. Finds open issues with the `claude` label (max 3 per run)
2. For each issue, runs Claude Code to implement the requested changes
3. Creates a pull request if successful
4. Updates issue labels and adds comments based on the outcome:
   - **Success**: Removes `claude` label, adds `pr-created` label, and comments with PR link
   - **Failure**: Adds `claude-error` label, keeps `claude` label for retry, and comments with error details

**Schedule:**
- Daily at 01:00, 03:00, and 05:00 JST (16:00, 18:00, 20:00 UTC previous day)

**Usage:**
1. Create an issue describing the task you want Claude to perform
2. Add the `claude` label to the issue
3. Wait for the next scheduled run, or manually trigger the workflow
4. Claude will process the issue and create a pull request if successful

#### 2. `claude-code-self-hosted.yml` - Manual Trigger via @claude

This workflow allows you to trigger Claude Code manually by mentioning `@claude` in issues or comments.

**Features:**
- Triggered when `@claude` is mentioned in:
  - New issue body
  - Issue comments
- Runs on self-hosted runner
- 30-minute timeout
- Processes the context where Claude was mentioned

**Usage:**
1. Create a new issue or comment on an existing issue
2. Include `@claude` in your message along with instructions
3. The workflow will trigger automatically
4. Claude will process your request

**Example:**
```
@claude Please add unit tests for the authentication module
```

### Disabled Workflows

The following workflows are currently disabled (`.disabled` extension) but can be enabled by removing the extension:

#### `claude-code-review.yml.disabled` - Automated Code Review

A workflow for automated pull request reviews using Claude Code.

**Potential features:**
- Triggers on pull request creation or updates
- Reviews code quality, best practices, security, and test coverage
- Posts review comments on the pull request
- Can be filtered by PR author or file paths

#### `claude.yml.disabled` - General Claude Integration

A general-purpose Claude Code workflow for GitHub-hosted runners.

**Potential features:**
- Triggers on `@claude` mentions in various contexts:
  - Issue comments
  - Pull request review comments
  - Pull request reviews
  - New issues
- Read-only permissions (for review and analysis tasks)
- Runs on GitHub-hosted `ubuntu-latest` runners

## Setup Instructions

### Prerequisites

1. A GitHub repository
2. A self-hosted GitHub Actions runner configured and running
3. A Claude Code OAuth token

### Self-Hosted Runner Setup

1. **Install the GitHub Actions Runner:**
   Follow the [official GitHub documentation](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners) to install and configure a self-hosted runner for your repository.

2. **Configure the runner:**
   ```bash
   # Download and extract the runner
   mkdir actions-runner && cd actions-runner
   # Follow GitHub's instructions for your OS

   # Configure the runner
   ./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPO --token YOUR_TOKEN

   # Start the runner
   ./run.sh
   ```

3. **Set up as a service (optional but recommended):**
   ```bash
   sudo ./svc.sh install
   sudo ./svc.sh start
   ```

### Required Secrets

Add the following secret to your repository:

1. **`CLAUDE_CODE_OAUTH_TOKEN`** (Required)
   - Navigate to your repository Settings > Secrets and variables > Actions
   - Click "New repository secret"
   - Name: `CLAUDE_CODE_OAUTH_TOKEN`
   - Value: Your Claude Code OAuth token
   - How to obtain: Follow the [Claude Code authentication guide](https://github.com/anthropics/claude-code-action)

### Required Labels

Create the following labels in your repository:

- **`claude`** - Add this label to issues you want Claude to process automatically
- **`pr-created`** - Automatically added when Claude successfully creates a PR
- **`claude-error`** - Automatically added when Claude encounters an error

You can create these labels via:
```bash
gh label create claude --color 0E8A16 --description "Issues for Claude to process"
gh label create pr-created --color 1D76DB --description "PR created by Claude"
gh label create claude-error --color D93F0B --description "Claude processing error"
```

## How to Use the Auto-Scheduler Feature

The auto-scheduler is the easiest way to leverage Claude Code for automated development tasks.

### Step-by-Step Guide:

1. **Create an Issue:**
   - Go to the Issues tab
   - Click "New issue"
   - Write a clear, detailed description of what you want Claude to implement

2. **Add the `claude` Label:**
   - Add the `claude` label to your issue
   - This signals the auto-scheduler to process it

3. **Wait for Processing:**
   - The scheduler runs automatically at 01:00, 03:00, and 05:00 JST
   - Or trigger manually via Actions tab > Claude Auto Scheduler > Run workflow

4. **Review the Results:**
   - **If successful:** You'll receive a comment with a link to the created PR
   - **If failed:** You'll receive a comment with error details and the workflow run link

5. **Review the Pull Request:**
   - Review the changes Claude made
   - Test the implementation
   - Merge when satisfied

### Best Practices for Issue Descriptions:

- Be specific and detailed about what you want
- Include context about the codebase structure if relevant
- Specify any particular requirements or constraints
- Break down complex tasks into multiple smaller issues

### Example Issue:

```markdown
Title: Add error logging to API endpoints

Description:
Please add comprehensive error logging to all API endpoints in the `src/api/` directory.

Requirements:
1. Use the existing logger utility from `src/utils/logger.ts`
2. Log errors with appropriate severity levels
3. Include request context (method, path, user ID if available)
4. Don't log sensitive information like passwords or tokens
```

## Allowed Tools

The workflows are configured with the following allowed tools for security:
- `Write`, `Edit`, `Read` - File operations
- `Glob`, `Grep` - File searching
- `Bash(git:*)` - Git operations
- `Bash(gh:*)` - GitHub CLI operations
- `Bash(ls:*)`, `Bash(pwd:*)`, `Bash(cd:*)` - Basic shell navigation
- `Bash(echo:*)`, `Bash(jq:*)` - Utility commands

## Workflow Permissions

The workflows run with the following permissions:
- `contents: write` - Create branches and commits
- `pull-requests: write` - Create and update pull requests
- `issues: write` - Update issues and labels
- `id-token: write` - OIDC token for authentication
- `actions: read` - Read workflow run information

## Troubleshooting

### Issues Not Being Processed

- Verify the `claude` label is correctly applied
- Check that the self-hosted runner is online and healthy
- Review the workflow run logs in the Actions tab
- Ensure the `CLAUDE_CODE_OAUTH_TOKEN` secret is set correctly

### Pull Requests Not Created

- Check the workflow run logs for errors
- Verify repository permissions allow the runner to create PRs
- Ensure the issue description is clear and actionable
- Look for the `claude-error` label and associated comment for details

### Self-Hosted Runner Issues

- Ensure the runner service is running: `sudo ./svc.sh status`
- Check runner logs for connection issues
- Verify network connectivity to GitHub
- Restart the runner service if needed: `sudo ./svc.sh restart`

## Contributing

This is a testing repository. Feel free to create issues with the `claude` label to test the automated workflows!

## License

This repository is for testing and demonstration purposes.

## Resources

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Claude Code Action GitHub Repository](https://github.com/anthropics/claude-code-action)
- [GitHub Actions Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners)
