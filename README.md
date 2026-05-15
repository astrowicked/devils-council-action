# Devils Council GitHub Action

> Adversarial AI review of plans and code — 4 expert personas find what you missed before you ship.

Posts review findings as a PR comment. Catches architectural gaps, security issues, over-engineering, and missing operational details automatically on every PR.

## Usage

```yaml
name: Plan Review
on:
  pull_request:
    paths:
      - '.planning/**'
      - 'docs/**/*.md'

jobs:
  council-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4

      - uses: astrowicked/devils-council-action@v1
        with:
          artifact: '.planning/**/PLAN.md'
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `artifact` | Yes | — | Path/glob to the plan file(s) to review |
| `anthropic-api-key` | Yes | — | Anthropic API key for persona agents |
| `model` | No | `claude-sonnet-4-20250514` | Model for review |
| `github-token` | No | `${{ github.token }}` | Token for PR comments |
| `severity-threshold` | No | `high` | Minimum severity to show |
| `max-findings` | No | `15` | Max findings in PR comment |

## Outputs

| Output | Description |
|--------|-------------|
| `blocker-count` | Number of BLOCKER findings |
| `high-count` | Number of HIGH findings |
| `synthesis-path` | Path to SYNTHESIS.md |

## What It Catches

The council spawns 4 expert personas that independently review your plan:

- **Staff Engineer** — over-engineering, wrong abstractions, YAGNI violations
- **SRE** — missing runbooks, hand-wavy mitigations, operational gaps
- **Product Manager** — scope creep, missing business justification, no success criteria
- **Devil's Advocate** — circular reasoning, unstated assumptions, false confidence

See [real-world examples](https://github.com/astrowicked/devils-council/tree/main/examples) of what the council catches.

## PR Comment Format

The action posts (or updates) a single PR comment with:
- Severity summary table (blockers + highs)
- Collapsible full synthesis with per-persona findings
- Links to the plugin for local use

## License

MIT
