# GitStats Action

[![GitHub Marketplace](https://img.shields.io/badge/GitHub_Marketplace-gitstats--action-blue.svg)](https://github.com/marketplace/actions/gitstats-action)

A GitHub Action that generates insightful visual reports from Git repositories using [GitStats](https://github.com/shenxianpeng/gitstats).

Powered by GitStats v2 — modern terminal-inspired UI, interactive Chart.js charts, zero system dependencies.

## Quick Start

```yaml
name: Generate GitStats Report

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday at midnight

jobs:
  gitstats:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Fetch all history for accurate stats

      - name: Generate GitStats Report
        uses: shenxianpeng/gitstats-action@v1
        with:
          output: gitstats-report

      - name: Upload Report as Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: gitstats-report
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to the git repository | No | `.` |
| `output` | Output directory for the report | No | `gitstats-report` |
| `project_name` | Project name shown in the report | No | (repo dir name) |
| `commit_begin` | Start of commit range (e.g. `10` for last 10 commits) | No | (all commits) |
| `commit_end` | End of commit range | No | `HEAD` |
| `start_date` | Starting date for commits (`YYYY-MM-DD`) | No | (no limit) |
| `end_date` | Ending date for commits (`YYYY-MM-DD`) | No | (no limit) |
| `config` | Additional config overrides (comma-separated `key=value`) | No | |
| `ai_enabled` | Enable AI-powered summaries | No | `false` |
| `ai_provider` | AI provider: `openai`, `claude`, `gemini`, `ollama` | No | |
| `ai_model` | AI model (e.g. `gpt-4`, `claude-3-5-sonnet-20241022`) | No | |

## Outputs

| Output | Description |
|--------|-------------|
| `report_path` | Path to the generated report directory |

## Examples

### Basic Usage

```yaml
- uses: shenxianpeng/gitstats-action@v1
  with:
    output: report
```

### Filter by Date Range

```yaml
- uses: shenxianpeng/gitstats-action@v1
  with:
    output: report
    start_date: '2024-01-01'
    end_date: '2024-12-31'
```

### Custom Config Overrides

```yaml
- uses: shenxianpeng/gitstats-action@v1
  with:
    output: report
    project_name: My Project
    config: max_authors=15,exclude_exts=png,jpg,svg
```

### AI-Powered Report

```yaml
- uses: shenxianpeng/gitstats-action@v1
  with:
    output: report
    ai_enabled: 'true'
    ai_provider: openai
    ai_model: gpt-4
  env:
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

### Deploy to GitHub Pages

```yaml
name: Deploy GitStats to Pages

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
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: shenxianpeng/gitstats-action@v1
        with:
          output: gitstats-report

      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: gitstats-report
      - uses: actions/deploy-pages@v4
        id: deployment
```

## What's Included

- **General**: total files, lines, commits, authors, age
- **Activity**: commits by hour of day, day of week, month of year, year
- **Authors**: list of authors (commits, first/last commit date, age), author of month/year
- **Files**: file count by date, extensions, file churn
- **Lines**: line of code by date
- **Tags**: tags by date and author
- **AI Insights** (optional): natural language summaries powered by OpenAI / Claude / Gemini

## How It Works

Composite action — runs directly on the runner, no Docker overhead. On first use it installs [gitstats](https://pypi.org/project/gitstats/) via pip.

## Requirements

- `actions/checkout@v4` with `fetch-depth: 0` **must** be run before this action to fetch full git history
- Runner: `ubuntu-latest`

## License

MIT © Xianpeng Shen
