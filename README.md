# Claude PR Documentation Action

Automatically generate comprehensive documentation for your merged pull requests using Claude AI. This GitHub Action helps maintain consistent and detailed documentation by analyzing PR changes and creating well-structured markdown files.

## Features

- 🤖 **AI-Powered Documentation**: Uses Claude AI to analyze code changes and generate meaningful documentation
- ⚙️ **Highly Configurable**: Customize thresholds, prompts, and output directories
- 🔄 **Automatic PR Creation**: Creates a new PR with the generated documentation
- 📊 **Smart Filtering**: Only documents PRs that meet your criteria (lines/files changed)
- 🏷️ **Customizable Tags**: Add custom tags to commits and PR titles
- 🚀 **Easy Integration**: Simple setup with sensible defaults

## Usage

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
      - uses: your-username/claude-pr-documentation@v1
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
    if: |
      github.event.pull_request.merged == true &&
      !contains(github.event.pull_request.labels.*.name, 'skip-docs')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - uses: your-username/claude-pr-documentation@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          min_lines_changed: '100'
          min_files_changed: '3'
          documentation_directory: 'docs/pull-requests'
          commit_tags: '[skip ci] [auto-doc]'
          pr_title_tags: '[Documentation]'
          timeout_minutes: '15'
          documentation_prompt: |
            Analyze PR #${{ github.event.pull_request.number }} and create detailed documentation.
            
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
| `anthropic_api_key` | Anthropic API key for Claude | Yes | - |
| `min_lines_changed` | Minimum lines changed to trigger documentation | No | `50` |
| `min_files_changed` | Minimum files changed to trigger documentation | No | `1` |
| `documentation_prompt` | Custom prompt for Claude | No | See [default prompt](#default-prompt) |
| `commit_tags` | Tags to append to commit message | No | `[skip ci]` |
| `pr_title_tags` | Tags to append to PR title | No | `[skip ci]` |
| `documentation_directory` | Directory for documentation files | No | `docs/prs` |
| `timeout_minutes` | Timeout for Claude (minutes) | No | `10` |
| `github_token` | GitHub token for creating PR | No | `${{ github.token }}` |

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

## Requirements

- GitHub repository with pull requests enabled
- Anthropic API key (get one at [console.anthropic.com](https://console.anthropic.com))
- Proper repository permissions (contents: write, pull-requests: write)

## Security Considerations

- Store your Anthropic API key as a GitHub secret
- The action only has access to your repository content
- Generated documentation is reviewed through the PR process

## Filtering and Exclusions

The action automatically skips documentation for:
- Unmerged PRs
- PRs created by bots
- PRs with existing documentation
- PRs below the configured thresholds

You can add additional filters in your workflow configuration.

## Troubleshooting

### Documentation not generated

1. Check if the PR meets the minimum thresholds
2. Verify the Anthropic API key is correctly set
3. Check the workflow logs for specific errors

### Empty or incomplete documentation

1. Increase the `timeout_minutes` value
2. Simplify the `documentation_prompt`
3. Ensure the PR has meaningful changes to analyze

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built with [Claude Code Action](https://github.com/anthropics/claude-code-action)
- Inspired by the need for better PR documentation practices