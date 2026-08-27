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

### Add a badge to your README

Every report ships with a `badge.svg` deployed right next to it — a shields.io-style badge in the gitstats brand colors showing your repository's **live commit count**. It updates automatically on every deploy.

Live example, served from the [gitstats demo report](https://shenxianpeng.github.io/gitstats/) (click one):

[![GitStats report](https://shenxianpeng.github.io/gitstats/badge.svg)](https://shenxianpeng.github.io/gitstats/) [![GitStats last commit](https://shenxianpeng.github.io/gitstats/badges/last-commit.svg)](https://shenxianpeng.github.io/gitstats/)

After the workflow runs, open the job summary: it contains ready-to-copy badge markdown for your repository. It looks like this:

```markdown
[![GitStats](https://<owner>.github.io/<repo>/badge.svg)](https://<owner>.github.io/<repo>/)
```

Anyone who sees the badge can click it to jump straight to your full GitStats report. The badge URL is also available programmatically via the `badge_url` and `badge_markdown` outputs.

**Customizing the badge** — every deploy also publishes a `badges/` directory with one pre-rendered badge per metric, so a different metric is just a different URL: `badges/commits.svg`, `badges/last-commit.svg` (date of the latest commit, e.g. `Aug 2026`), `badges/authors.svg`, `badges/files.svg`, `badges/lines.svg`. Label, color, and shape are controlled through the `config` input:

```yaml
- uses: shenxianpeng/gitstats-action@v0.1.1
  with:
    deploy-to-pages: true
    config: badge_metric=last-commit|badge_color=green|badge_style=flat-square
```

For full shields.io URL-parameter customization (any color, `style=for-the-badge`, logos), each metric is also published as `badges/<metric>.json` in the [shields endpoint schema](https://shields.io/badges/endpoint-badge):

```markdown
[![GitStats](https://img.shields.io/endpoint?url=https://<owner>.github.io/<repo>/badges/commits.json&style=for-the-badge)](https://<owner>.github.io/<repo>/)
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to the git repository | No | `.` |
| `output` | Output directory for the report | No | `gitstats-report` |
| `config` | GitStats config overrides (pipe-separated `key=value`). All [gitstats options](https://github.com/shenxianpeng/gitstats#configuration) supported | No | |
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
| `badge_url` | URL of the deployed `badge.svg` (only set when `deploy-to-pages` is `true`) |
| `badge_markdown` | Ready-to-copy README badge markdown linking to the report (only set when `deploy-to-pages` is `true`) |

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
    config: start_date=2024-01-01|end_date=2024-12-31
```

### Custom Config Overrides

```yaml
- uses: shenxianpeng/gitstats-action@v0.1.1
  with:
    output: report
    config: project_name=My Project|max_authors=15|exclude_exts=png,jpg,svg
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

## Common Config Options

Here are some commonly used gitstats configuration keys you can pass via the `config` input:

| Key | Example | Description |
|-----|---------|-------------|
| `project_name` | `My Project` | Project name shown in the report |
| `commit_begin` | `10` | Last N commits only |
| `commit_end` | `abc123` | End commit hash |
| `start_date` | `2024-01-01` | Starting date |
| `end_date` | `2024-12-31` | Ending date |
| `max_authors` | `20` | Max authors to display |
| `exclude_exts` | `png,jpg,svg` | Exclude file extensions |
| `linenos` | `true` | Show line numbers in code |

See the [gitstats documentation](https://github.com/shenxianpeng/gitstats#configuration) for the full list.

## What's Included

- **General**: total files, lines, commits, authors, age
- **Activity**: commits by hour of day, day of week, month of year, year
- **Authors**: list of authors (commits, first/last commit date, age), author of month/year
- **Files**: file count by date, extensions, file churn
- **Lines**: line of code by date
- **Tags**: tags by date and author
- **Badge**: a shareable `badge.svg` with live commit count, linking readers to the report
- **AI Insights** (optional): natural language summaries powered by OpenAI / Claude / Gemini / Ollama

## License

MIT — see [LICENSE](./LICENSE).
