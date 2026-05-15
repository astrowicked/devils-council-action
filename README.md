# Devils Council GitHub Action

> Adversarial AI review of plans and code — 4 expert personas find what you missed before you ship.

Posts review findings as a PR comment. Catches architectural gaps, security issues, over-engineering, and missing operational details automatically on every PR.

## Usage

### Anthropic (direct API)

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

### AWS Bedrock

```yaml
      - uses: astrowicked/devils-council-action@v1
        with:
          artifact: '.planning/**/PLAN.md'
          provider: bedrock
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
```

Or with OIDC (recommended for GitHub-hosted runners with AWS role assumption):

```yaml
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-dc-review
          aws-region: us-east-1

      - uses: astrowicked/devils-council-action@v1
        with:
          artifact: '.planning/**/PLAN.md'
          provider: bedrock
          # No AWS keys needed — uses ambient credentials from configure-aws-credentials
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `artifact` | Yes | — | Path/glob to the plan file(s) to review |
| `provider` | No | `anthropic` | LLM provider: `anthropic` or `bedrock` |
| `model` | No | auto | Model ID (defaults: `claude-sonnet-4-20250514` for anthropic, `us.anthropic.claude-sonnet-4-20250514-v1:0` for bedrock) |
| `anthropic-api-key` | If anthropic | — | Anthropic API key |
| `aws-access-key-id` | If bedrock* | — | AWS access key (skip if using OIDC) |
| `aws-secret-access-key` | If bedrock* | — | AWS secret key (skip if using OIDC) |
| `aws-region` | No | `us-east-1` | AWS region for Bedrock |
| `github-token` | No | `${{ github.token }}` | Token for PR comments |
| `severity-threshold` | No | `high` | Minimum severity to show |
| `max-findings` | No | `15` | Max findings in PR comment |

*For Bedrock, you can use `aws-actions/configure-aws-credentials` with OIDC instead of passing keys directly.

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
