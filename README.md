# GitStats Action

[![Test](https://github.com/shenxianpeng/gitstats-action/actions/workflows/test.yml/badge.svg)](https://github.com/shenxianpeng/gitstats-action/actions/workflows/test.yml) [![GitHub Marketplace](https://img.shields.io/badge/GitHub_Marketplace-gitstats--action-blue.svg)](https://github.com/marketplace/actions/gitstats-action)

A GitHub Action that generates insightful visual reports from Git repositories using [gitstats](https://github.com/shenxianpeng/gitstats).

> [!TIP]
> Pin to a specific version for stability: `shenxianpeng/gitstats-action@v0.1.1`.
> Set up an [annotated tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) `v1` pointing to your latest release so users can use `@v1` for automatic minor updates.

## Quick Start

### One-liner: Generate + Deploy to GitHub Pages

```yaml
name: GitStats Report

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  gitstats:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - uses: shenxianpeng/gitstats-action@v0.1.1
        with:
          deploy-to-pages: true
```

That's it. One `uses` line, and your report is live on GitHub Pages.

> 💡 Make sure **GitHub Pages** is enabled in your repo settings (Settings → Pages → Source: "Deploy from a branch" → Branch: `gh-pages`).

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to the git repository | No | `.` |
| `output` | Output directory for the report | No | `gitstats-report` |
| `project-name` | Project name shown in the report | No | (repo dir name) |
| `commit-start` | Start of commit range (e.g. `10` for last 10 commits) | No | (all commits) |
| `commit-end` | End of commit range (defaults to `HEAD` in gitstats when not set) | No | (all commits until HEAD) |
| `start-date` | Starting date for commits (`YYYY-MM-DD`) | No | (no limit) |
| `end-date` | Ending date for commits (`YYYY-MM-DD`) | No | (no limit) |
| `config` | Additional config overrides (pipe-separated `key=value`) | No | |
| `ai-enabled` | Enable AI-powered summaries | No | `false` |
| `ai-provider` | AI provider: `openai`, `claude`, `gemini`, `ollama` | No | |
| `ai-model` | AI model (e.g. `gpt-4o`, `claude-sonnet-4-20250514`) | No | |
| `deploy-to-pages` | Automatically deploy the report to GitHub Pages | No | `false` |
| `deploy-branch` | Branch to deploy to (only used when `deploy-to-pages` is `true`) | No | `gh-pages` |

## Outputs

| Output | Description |
|--------|-------------|
| `report_path` | Path to the generated report directory |
| `pages_url` | URL of the deployed GitHub Pages site (only set when `deploy-to-pages` is `true`) |

## Examples

### Just generate (no deployment)

```yaml
- uses: shenxianpeng/gitstats-action@v0.1.1
  with:
    output: report
```

### Filter by Date Range

```yaml
- uses: shenxianpeng/gitstats-action@v0.1.1
  with:
    output: report
    start-date: '2024-01-01'
    end-date: '2024-12-31'
```

### Custom Config Overrides

```yaml
- uses: shenxianpeng/gitstats-action@v0.1.1
  with:
    output: report
    project-name: My Project
    config: max_authors=15|exclude_exts=png,jpg,svg
```

### AI-Powered Report

```yaml
- uses: shenxianpeng/gitstats-action@v0.1.1
  with:
    output: report
    ai-enabled: 'true'
    ai-provider: openai
    ai-model: gpt-4o
  env:
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

### Custom Deploy Branch

```yaml
- uses: shenxianpeng/gitstats-action@v0.1.1
  with:
    output: report
    deploy-to-pages: true
    deploy-branch: my-pages-branch
```

### Manual Deploy (advanced)

If you need more control over the deployment (e.g., custom environment, deployment URL output), use the official Pages actions:

```yaml
name: Deploy gitstats to pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0

      - uses: shenxianpeng/gitstats-action@v0.1.1
        with:
          output: gitstats-report

      - uses: actions/configure-pages@v6
      - uses: actions/upload-pages-artifact@v5
        with:
          path: gitstats-report
      - uses: actions/deploy-pages@v5
        id: deployment
```

## What's Included

- **General**: total files, lines, commits, authors, age
- **Activity**: commits by hour of day, day of week, month of year, year
- **Authors**: list of authors (commits, first/last commit date, age), author of month/year
- **Files**: file count by date, extensions, file churn
- **Lines**: line of code by date
- **Tags**: tags by date and author
- **AI Insights** (optional): natural language summaries powered by OpenAI / Claude / Gemini / Ollama

## License

MIT — see [LICENSE](./LICENSE).
