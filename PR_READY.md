# Pull Request Ready for Issue #3

This branch `docs/add-comprehensive-readme` is ready to be merged and contains the comprehensive README documentation requested in Issue #3.

## To create the pull request manually:

1. Visit: https://github.com/SH11235/test-run-claude-code-actions/compare/main...docs/add-comprehensive-readme

2. Click "Create pull request"

3. Use this title:
   ```
   Add comprehensive README documentation
   ```

4. Use this description:
   ```markdown
   ## Summary

   This PR adds a comprehensive README.md file to document the Claude Code Actions testing repository with self-hosted runner configuration.

   ## Changes

   - ✅ Created detailed README.md with complete documentation
   - ✅ Documented active workflows:
     - `claude-auto-scheduler.yml` - Scheduled issue processing with label-based triggering
     - `claude-code-self-hosted.yml` - Manual `@claude` mention-based triggering
   - ✅ Documented disabled workflows:
     - `claude-code-review.yml.disabled` - Automated PR review workflow
     - `claude.yml.disabled` - General-purpose GitHub-hosted workflow
   - ✅ Added comprehensive setup instructions for self-hosted runners
   - ✅ Documented the auto-scheduler feature with usage examples
   - ✅ Listed required secrets (`CLAUDE_CODE_OAUTH_TOKEN`)
   - ✅ Included troubleshooting tips and resource links
   - ✅ Documented allowed tools configuration and permissions

   ## Key Documentation Sections

   1. **Workflows** - Detailed explanation of each workflow file, their triggers, features, and usage
   2. **Setup Instructions** - Step-by-step guide for setting up self-hosted runners
   3. **Required Secrets** - Documentation of necessary authentication tokens
   4. **Auto-Scheduler Usage** - How to use the `claude` label to queue issues
   5. **Allowed Tools** - Security configuration and tool restrictions
   6. **Troubleshooting** - Common issues and solutions

   ## Testing

   - [x] README renders correctly on GitHub
   - [x] All links are valid
   - [x] Documentation accurately reflects workflow configurations
   - [x] Setup instructions are complete and accurate

   Fixes #3

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   ```

## What was completed:

✅ Read existing workflow files in `.github/workflows/`
✅ Analyzed workflow purposes and configurations
✅ Created comprehensive README.md documentation covering:
  - Repository purpose (Claude Code Actions testing with self-hosted runner)
  - Active workflows (claude-auto-scheduler.yml and claude-code-self-hosted.yml)
  - Disabled workflows (claude-code-review.yml.disabled and claude.yml.disabled)
  - Self-hosted runner setup instructions
  - Auto-scheduler feature usage (adding 'claude' label)
  - Required secrets (CLAUDE_CODE_OAUTH_TOKEN)
  - Usage examples and troubleshooting

## Branch Information:

- Branch name: `docs/add-comprehensive-readme`
- Base branch: `main`
- Commit: Contains README.md with full documentation

The README.md file is ready and pushed to the remote branch. A pull request just needs to be created manually due to GitHub Actions token limitations.
