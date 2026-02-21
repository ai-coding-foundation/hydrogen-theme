---
name: hatch3r-gh-agentic-workflows
description: Set up GitHub Agentic Workflows for continuous AI-powered repository automation
---

# GitHub Agentic Workflows Integration

GitHub Agentic Workflows (technical preview, Feb 2026) bring AI agent orchestration into
GitHub Actions. This skill guides setup for hatch3r-managed projects.

## Overview

Agentic Workflows are markdown files in `.github/workflows/` with YAML frontmatter that
compile to GitHub Actions jobs. They support multiple AI engines (GitHub Copilot, Claude,
OpenAI Codex) and use MCP for tool access.

## Available Workflow Templates

hatch3r recommends these agentic workflow patterns for projects:

### 1. Continuous Test Improvement

Automatically assess test coverage and add high-value tests.

```yaml
# .github/workflows/hatch3r-continuous-testing.md
---
name: Continuous Test Improvement
on:
  schedule:
    - cron: '0 6 * * 1'
  workflow_dispatch:
engine: copilot
permissions:
  contents: read
  pull-requests: write
---
```

Analyze test coverage gaps and open PRs with new tests for uncovered critical paths.

### 2. Continuous Triage

Automatically summarize, label, and route new issues.

```yaml
# .github/workflows/hatch3r-continuous-triage.md
---
name: Continuous Triage
on:
  issues:
    types: [opened]
engine: copilot
permissions:
  issues: write
---
```

When a new issue is opened, analyze it, apply labels from the hatch3r taxonomy
(`type:*`, `priority:*`, `area:*`), and add a triage summary comment.

### 3. Continuous Documentation

Keep documentation aligned with code changes.

```yaml
# .github/workflows/hatch3r-continuous-docs.md
---
name: Continuous Documentation
on:
  pull_request:
    types: [closed]
    branches: [main]
engine: copilot
permissions:
  contents: write
  pull-requests: write
---
```

After a PR is merged, check if documentation needs updating and open a follow-up PR.

## Security Considerations

- Workflows run in sandboxed environments with minimal permissions
- Use read-only defaults; only grant write permissions when needed
- Review all AI-generated changes before merging
- Network isolation and tool allowlisting are enforced by the runtime

## Integration with hatch3r

- hatch3r's label taxonomy (`type:*`, `executor:*`, `priority:*`) aligns with agentic triage
- The hatch3r-test-writer agent's patterns can inform continuous testing workflows
- The hatch3r-docs-writer agent's patterns can inform continuous documentation
- Board management commands complement continuous triage

## Setup

1. Enable GitHub Agentic Workflows in your repository settings
2. Create workflow files in `.github/workflows/` using the templates above
3. Configure the AI engine (copilot is default, claude and codex are alternatives)
4. Set appropriate permissions for each workflow
5. Monitor workflow runs in the Actions tab
