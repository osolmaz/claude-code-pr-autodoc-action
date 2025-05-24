---
author: Claude
date: 2025-01-24
title: Building the Claude Code PR Documentation Action
---

# Building the Claude Code PR Documentation Action

## Project Overview

This document chronicles the development of the Claude Code PR Documentation Action by TextCortex, a GitHub Action that automatically generates comprehensive documentation for merged pull requests using Claude Code (by Anthropic). The action was created to solve a common problem in software development: maintaining up-to-date, meaningful documentation for code changes.

## Motivation

### The Problem

In modern software development, pull requests often contain significant changes that impact multiple aspects of a system. While developers write PR descriptions, these are often brief and focused on immediate context. Over time, understanding why certain changes were made, their technical implementation details, and their broader impact becomes challenging. This is especially true for:

- Onboarding new team members who need to understand historical decisions
- Debugging issues that may stem from past changes
- Conducting post-mortems or architectural reviews
- Maintaining compliance and audit trails

### The Solution

The Claude Code PR Documentation Action addresses these challenges by:

1. **Automatically triggering** when PRs are merged to specified branches
2. **Analyzing the complete changeset** using git diff and commit history
3. **Generating structured documentation** with consistent formatting
4. **Creating a new PR** with the documentation for review
5. **Maintaining a searchable archive** of all significant changes

## Development Journey

### Initial Setup and Branding Alignment

The project began as a generic "Claude AI" action but was realigned to be part of the Claude Code ecosystem. This involved:

1. **Rebranding from Claude AI to Claude Code**

   - Updated all references throughout the codebase
   - Changed bot attribution from "Claude[bot]" to "Claude Code[bot]"
   - Aligned with the official Claude Code Action patterns

2. **Integration with Claude GitHub App**
   - Leveraged existing Claude GitHub app infrastructure
   - Ensured compatibility for users who already have Claude Code installed
   - Maintained consistent security model and permissions

### Key Technical Challenges and Solutions

#### 1. GitHub Actions Template Syntax Limitations

**Challenge**: GitHub Actions composite actions don't allow using `${{ inputs.variable }}` within input default values.

**Solution**:

- Used placeholder values in the default prompt
- Created a dynamic prompt preparation step that replaces placeholders at runtime
- Implemented string substitution: `PROMPT="${PROMPT//docs\/prs/${{ inputs.documentation_directory }}}"`

#### 2. Handling Merged PR References

**Challenge**: GitHub removes `refs/pull/*/merge` references after PRs are merged, causing git fetch failures.

**Solution**:

```bash
# Try multiple approaches to fetch PR commits
git fetch origin +refs/pull/${{ github.event.pull_request.number }}/merge:refs/remotes/origin/pr/${{ github.event.pull_request.number }} 2>/dev/null || true

# For merged PRs, use the merge commit
if [ "${{ github.event.pull_request.merged }}" == "true" ] && [ -n "$MERGE_SHA" ]; then
  MERGE_PARENT=$(git rev-parse ${MERGE_SHA}^1 2>/dev/null || echo "")
  # Use merge commit for accurate diff
fi
```

#### 3. Preventing Infinite Documentation Loops

**Challenge**: Documentation PRs could trigger more documentation PRs, creating an infinite loop.

**Solution**: Implemented multi-layer protection:

1. **Workflow-level filtering** (prevents container spin-up):

   ```yaml
   if: |
     github.event.pull_request.merged == true &&
     github.event.pull_request.user.type != 'Bot' &&
     !startsWith(github.event.pull_request.title, 'docs: Add documentation for PR')
   ```

2. **Action-level checks** (backup protection):
   - Skip PRs created by claude[bot] or claude-code[bot]
   - Skip PRs with "automated" or "documentation" labels
   - Skip PRs with documentation-specific titles

#### 4. Robust Line Counting

**Challenge**: The initial regex-based approach for counting changed lines failed when PRs had only insertions or only deletions.

**Solution**: Replaced with an awk script that handles all cases:

```bash
LINES_CHANGED=$(git diff --shortstat ${BASE_SHA}...${HEAD_SHA} | \
  awk '{ins=0; del=0; for(i=1;i<=NF;i++){if($i ~ /insertion/) ins=$(i-1); if($i ~ /deletion/) del=$(i-1)} print ins+del}' || echo "0")
```

### Cloud Provider Support

The action was enhanced to support multiple Claude providers:

1. **Direct Anthropic API** (default)
2. **AWS Bedrock** with OIDC authentication
3. **Google Vertex AI** with OIDC authentication

Each provider requires specific:

- Model naming conventions
- Authentication flows
- Permission configurations

### Security Considerations

1. **API Key Protection**

   - Enforced use of GitHub Secrets
   - Clear warnings against hardcoding keys
   - Documentation emphasizes security best practices

2. **GitHub App Permissions**

   - Minimal required permissions
   - Scoped to repository only
   - Short-lived tokens

3. **Commit Signing**
   - All commits automatically signed
   - Proper attribution with Co-Authored-By

## Architecture Decisions

### Composite Action vs. JavaScript/Docker

Chose composite action for:

- Simplicity and maintainability
- Direct use of existing Claude Code Action
- No additional runtime requirements
- Easy debugging and modification

### File Organization

```
/
├── action.yml              # Main action definition
├── README.md              # User-facing documentation
├── LICENSE                # MIT License
├── examples/              # Example workflows
│   ├── claude-pr-documentation.yml
│   ├── claude-pr-documentation-advanced.yml
│   ├── claude-pr-documentation-bedrock.yml
│   └── claude-pr-documentation-vertex.yml
└── docs/                  # Project documentation
    └── 2025-01-24-building-claude-code-pr-documentation-action.md
```

### Configuration Philosophy

- **Sensible defaults**: Works out-of-the-box with minimal configuration
- **Progressive disclosure**: Basic usage is simple, advanced features available
- **Flexibility**: Customizable prompts, thresholds, and tools
- **Safety first**: Built-in protections against common pitfalls

## Key Features Implemented

### 1. Smart Filtering

- Minimum lines/files changed thresholds
- Label-based exclusions
- Bot detection
- Title pattern matching

### 2. Flexible Documentation

- Customizable output directory
- YAML frontmatter for metadata
- Consistent naming format
- Custom instructions support

### 3. Robust Git Operations

- Multiple fallback strategies for commits
- Merge commit handling
- Branch management
- Clean working directory

### 4. Claude Code Integration

- Full tool access (with restrictions)
- Custom allowed/disallowed tools
- Model selection
- Timeout configuration

## Usage Patterns

### Basic Usage

For teams wanting automatic documentation with minimal setup:

```yaml
- uses: textcortex/claude-code-pr-autodoc-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Advanced Usage

For teams with specific requirements:

- Custom documentation directories by date
- Specific change thresholds
- Custom instructions for domain-specific documentation
- Integration with existing CI/CD pipelines

### Enterprise Usage

With Bedrock or Vertex AI:

- OIDC authentication
- Cross-region inference
- Compliance with data residency requirements

## Lessons Learned

1. **GitHub Actions Limitations**: Understanding platform constraints early saves debugging time
2. **Git Edge Cases**: Merged PRs behave differently than open PRs in Git
3. **User Experience**: Clear error messages and fallbacks improve adoption
4. **Security by Default**: Make the secure path the easy path
5. **Documentation**: Comprehensive examples are more valuable than lengthy explanations

## Future Considerations

While not implemented, potential enhancements could include:

1. **Webhook Support**: Direct integration with GitHub webhooks
2. **Multiple Output Formats**: Markdown, JSON, HTML
3. **Template System**: User-defined documentation templates
4. **Metrics Collection**: Track documentation coverage over time
5. **Integration with Documentation Sites**: Auto-publish to docs platforms

## Impact

This action enables teams to:

- Build comprehensive documentation archives automatically
- Reduce onboarding time for new developers
- Maintain better compliance and audit trails
- Understand the evolution of their codebase
- Focus on building features rather than documenting them manually

By leveraging Claude Code's understanding of code and context, the action bridges the gap between code changes and human-readable documentation, making software development more transparent and maintainable.

---

_This document itself was generated using Claude Code, demonstrating the kind of comprehensive documentation the action can produce._

## Attribution

This action was developed by TextCortex and is built on top of the Claude Code Action by Anthropic. While Claude Code and the underlying AI technology are created and maintained by Anthropic, this specific GitHub Action for PR documentation is a TextCortex project.
