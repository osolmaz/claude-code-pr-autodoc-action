# Claude Code PR Documentation Action

Automatically generate comprehensive documentation for your merged pull requests using [Claude Code](https://claude.ai/code). This GitHub Action helps maintain consistent and detailed documentation by analyzing PR changes and creating well-structured markdown files.

## Features

- 🤖 **AI-Powered Documentation**: Uses Claude Code to analyze code changes and generate meaningful documentation
- ⚙️ **Highly Configurable**: Customize thresholds, prompts, and output directories
- 🔄 **Automatic PR Creation**: Creates a new PR with the generated documentation
- 📊 **Smart Filtering**: Only documents PRs that meet your criteria (lines/files changed)
- 🏷️ **Customizable Tags**: Add custom tags to commits and PR titles
- 🚀 **Easy Integration**: Works seamlessly with the Claude GitHub app
- 🏃 **Runs on Your Infrastructure**: The action executes entirely on your own GitHub runner

## Quickstart

The easiest way to set up this action is if you already have the Claude GitHub app installed. If you've already used Claude Code Action, this will work automatically!

### Prerequisites

1. Install the Claude GitHub app to your repository: https://github.com/apps/claude
2. Add `ANTHROPIC_API_KEY` to your repository secrets ([Learn how to use secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions))

### Basic Usage

```yaml
name: Auto-generate PR Documentation

on:
  pull_request:
    types: [closed]
    branches:
      - main

jobs:
  generate-documentation:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - uses: anthropics/claude-code-pr-autodoc-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Advanced Usage

```yaml
name: Auto-generate PR Documentation

on:
  pull_request:
    types: [closed]
    branches:
      - main
      - develop

jobs:
  generate-documentation:
    # Only run for merged PRs that don't have the 'skip-docs' label
    # The action itself will also check for bot-created PRs to prevent loops
    if: |
      github.event.pull_request.merged == true &&
      !contains(github.event.pull_request.labels.*.name, 'skip-docs')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - uses: anthropics/claude-code-pr-autodoc-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          min_lines_changed: '100'
          min_files_changed: '3'
          documentation_directory: 'docs/pull-requests'
          commit_tags: '[skip ci] [auto-doc]'
          pr_title_tags: '[Documentation]'
          timeout_minutes: '15'
          custom_instructions: |
            Focus on:
            - Business impact and user-facing changes
            - Technical architecture decisions
            - Performance implications
            - Security considerations
            - Testing coverage
            
            Be concise but thorough.
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `anthropic_api_key` | Anthropic API key (required for direct API, not needed for Bedrock/Vertex) | No* | - |
| `min_lines_changed` | Minimum lines changed to trigger documentation | No | `50` |
| `min_files_changed` | Minimum files changed to trigger documentation | No | `1` |
| `documentation_prompt` | Custom prompt for Claude | No | See [default prompt](#default-prompt) |
| `commit_tags` | Tags to append to commit message | No | `[skip ci]` |
| `pr_title_tags` | Tags to append to PR title | No | `[skip ci]` |
| `documentation_directory` | Directory for documentation files | No | `docs/prs` |
| `timeout_minutes` | Timeout for Claude (minutes) | No | `10` |
| `github_token` | GitHub token for Claude to operate with. **Only include this if you're connecting a custom GitHub app of your own!** | No | - |
| `model` | Model to use (provider-specific format required for Bedrock/Vertex) | No | - |
| `use_bedrock` | Use Amazon Bedrock with OIDC authentication instead of direct Anthropic API | No | `false` |
| `use_vertex` | Use Google Vertex AI with OIDC authentication instead of direct Anthropic API | No | `false` |
| `allowed_tools` | Additional tools for Claude to use (the base GitHub tools will always be included) | No | "" |
| `disallowed_tools` | Tools that Claude should never use | No | "" |
| `custom_instructions` | Additional custom instructions to include in the prompt for Claude | No | "" |

\*Required when using direct Anthropic API (default and when not using Bedrock or Vertex)

### Default Prompt

The default prompt instructs Claude to:
1. Analyze the PR changes using git commands
2. Create a properly formatted markdown file with YAML frontmatter
3. Document purpose, key modifications, technical details, impact, and any breaking changes
4. Include related issues and follow-up tasks

## Outputs

The action creates:
- A markdown file in the specified directory with the format: `YYYY-MM-DD-descriptive-title.md`
- A new PR containing the documentation
- Proper git history linking back to the original PR

## Example Documentation Output

```markdown
---
author: Claude
date: 2024-01-15
title: Implement Redis Caching for API Endpoints
pr_number: 142
pr_author: developer123
---

# Implement Redis Caching for API Endpoints

## Purpose and Context

This PR introduces Redis caching to improve API response times and reduce database load...

## Key Modifications

1. **Added Redis client configuration** (`src/config/redis.js`)
   - Configured connection pooling
   - Implemented retry logic

2. **Created caching middleware** (`src/middleware/cache.js`)
   - TTL-based cache invalidation
   - Cache key generation strategy

[... continued ...]
```

## Cloud Providers

You can authenticate with Claude using any of these three methods:

1. Direct Anthropic API (default)
2. Amazon Bedrock with OIDC authentication
3. Google Vertex AI with OIDC authentication

### Using with AWS Bedrock

```yaml
# For AWS Bedrock with OIDC
- name: Configure AWS Credentials (OIDC)
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
    aws-region: us-west-2

- name: Generate GitHub App token
  id: app-token
  uses: actions/create-github-app-token@v2
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}

- uses: anthropics/claude-code-pr-autodoc-action@v1
  with:
    model: "anthropic.claude-3-7-sonnet-20250219-beta:0"
    use_bedrock: "true"
    github_token: ${{ steps.app-token.outputs.token }}
    # ... other inputs

permissions:
  id-token: write # Required for OIDC
```

### Using with Google Vertex AI

```yaml
# For GCP Vertex AI with OIDC
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
    service_account: ${{ secrets.GCP_SERVICE_ACCOUNT }}

- name: Generate GitHub App token
  id: app-token
  uses: actions/create-github-app-token@v2
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}

- uses: anthropics/claude-code-pr-autodoc-action@v1
  with:
    model: "claude-3-7-sonnet@20250219"
    use_vertex: "true"
    github_token: ${{ steps.app-token.outputs.token }}
    # ... other inputs

permissions:
  id-token: write # Required for OIDC
```

## Security

### ⚠️ ANTHROPIC_API_KEY Protection

**CRITICAL: Never hardcode your Anthropic API key in workflow files!**

Your ANTHROPIC_API_KEY must always be stored in GitHub secrets:

```yaml
# CORRECT ✅
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

# NEVER DO THIS ❌
anthropic_api_key: "sk-ant-api03-..." # Exposed and vulnerable!
```

### GitHub App Permissions

The [Claude Code GitHub app](https://github.com/apps/claude) requires these permissions:

- **Pull Requests**: Read and write to create PRs and push changes
- **Issues**: Read and write to respond to issues
- **Contents**: Read and write to modify repository files

### Commit Signing

All commits made by Claude through this action are automatically signed with commit signatures.

## Filtering and Exclusions

The action automatically skips documentation for:
- Unmerged PRs
- PRs created by bots (including Claude Code itself) - prevents infinite loops
- PRs with titles starting with "docs: Add documentation for PR"
- PRs with "automated" or "documentation" labels
- PRs below the configured thresholds (lines/files changed)

You can add additional filters in your workflow configuration using the `if` condition on the job.

## Troubleshooting

### Documentation not generated

1. Check if the PR meets the minimum thresholds
2. Verify the Anthropic API key is correctly set
3. Check the workflow logs for specific errors
4. Ensure the Claude GitHub app is installed on your repository

### Empty or incomplete documentation

1. Increase the `timeout_minutes` value
2. Simplify the `custom_instructions`
3. Ensure the PR has meaningful changes to analyze

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

🤖 Built with [Claude Code](https://claude.ai/code)