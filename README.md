# Devils Council GitHub Action

> Adversarial AI review of plans and code — 4 expert personas find what you missed before you ship.

Posts review findings as inline PR comments on the exact lines that need attention. Catches architectural gaps, security issues, over-engineering, and missing operational details automatically on every PR.

## Quick Start: Code Review on Every PR

```yaml
name: Code Review
on: [pull_request]

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
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

That's it. The action fetches the PR diff, runs the full council, and posts findings as inline review comments on the relevant lines. Blockers appear as `REQUEST_CHANGES`; majors and minors as `COMMENT`.

---

## Usage

### Review code (default — auto-fetches PR diff)

```yaml
- uses: astrowicked/devils-council-action@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    # artifact defaults to 'pr-diff' — auto-fetches the PR diff
    # post-as defaults to 'both' — inline comments + summary
```

### Review plans on PR

```yaml
- uses: astrowicked/devils-council-action@v1
  with:
    artifact: '.planning/**/PLAN.md'
    type: plan
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    post-as: comment  # summary only (plans don't map to lines)
```

### AWS Bedrock

```yaml
- uses: astrowicked/devils-council-action@v1
  with:
    provider: bedrock
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1
```

Or with OIDC (recommended):

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/github-actions-dc-review
    aws-region: us-east-1

- uses: astrowicked/devils-council-action@v1
  with:
    provider: bedrock
    # Uses ambient credentials from configure-aws-credentials
```

### Block merge on blockers

```yaml
- uses: astrowicked/devils-council-action@v1
  id: review
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}

- name: Fail on blockers
  if: steps.review.outputs.verdict == 'BLOCK'
  run: |
    echo "::error::Devils Council found ${{ steps.review.outputs.blocker-count }} blocker(s)"
    exit 1
```

---

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `artifact` | No | `pr-diff` | Path/glob to review, or `pr-diff` to auto-fetch the PR diff |
| `type` | No | `auto` | Artifact type: `auto`, `plan`, `code-diff`, `rfc` |
| `post-as` | No | `both` | How to post: `inline-review`, `comment`, `both` |
| `provider` | No | `anthropic` | LLM provider: `anthropic` or `bedrock` |
| `model` | No | auto | Model ID (defaults per provider) |
| `anthropic-api-key` | If anthropic | — | Anthropic API key |
| `aws-access-key-id` | If bedrock* | — | AWS access key (skip if using OIDC) |
| `aws-secret-access-key` | If bedrock* | — | AWS secret key (skip if using OIDC) |
| `aws-region` | No | `us-east-1` | AWS region for Bedrock |
| `github-token` | No | `${{ github.token }}` | Token for PR comments (`pull-requests: write` needed) |
| `severity-threshold` | No | `major` | Minimum severity to post: `blocker`, `major`, `minor`, `nit` |
| `max-findings` | No | `20` | Max findings to include |
| `exclude-paths` | No | `*.lock,...` | Comma-separated patterns to exclude from code review |

*For Bedrock, use `aws-actions/configure-aws-credentials` with OIDC instead of passing keys directly.

## Outputs

| Output | Description |
|--------|-------------|
| `blocker-count` | Number of BLOCKER findings |
| `major-count` | Number of MAJOR findings |
| `verdict` | Overall: `BLOCK`, `WARN`, or `PASS` |
| `synthesis-path` | Path to SYNTHESIS.md |
| `report-path` | Path to HTML report (if python3 available) |

## What It Catches

The council spawns 4 core personas + signal-triggered specialists:

- **Staff Engineer** — over-engineering, wrong abstractions, YAGNI violations
- **SRE** — missing runbooks, hand-wavy mitigations, operational gaps
- **Product Manager** — scope creep, missing business justification, no success criteria
- **Devil's Advocate** — circular reasoning, unstated assumptions, false confidence
- **Security Reviewer** (auto-triggers on auth/crypto code) — attack vectors, missing controls
- **Junior Engineer** (auto-triggers on code diffs) — comprehension failures, unclear code

## How Findings Appear

### Inline review comments (code review)

Each finding appears as an inline comment on the exact file and line, with severity badge, persona attribution, and the "ask" (what to do about it). Blockers trigger `REQUEST_CHANGES`.

### Summary comment (plans + fallback)

A single PR comment with severity table and collapsible full synthesis. Updated in-place on re-runs (no comment spam).

## Permissions

```yaml
permissions:
  contents: read        # Read repo files
  pull-requests: write  # Post review comments
```

## License

MIT
