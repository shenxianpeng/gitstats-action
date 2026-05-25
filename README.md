# GitStats Action

[![GitHub Marketplace](https://img.shields.io/badge/GitHub_Marketplace-gitstats--action-blue.svg)](https://github.com/marketplace/actions/gitstats-action)

A GitHub Action that generates comprehensive Git repository statistics reports using [GitStats](https://github.com/tomgi/gitstats).

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
| `branch` | Git branch to analyze | No | (current branch) |
| `commit_begin` | First commit to include | No | (first commit) |
| `commit_end` | Last commit to include | No | `HEAD` |
| `config_file` | Path to a gitstats config file | No | |

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

### Analyze a Specific Branch

```yaml
- uses: shenxianpeng/gitstats-action@v1
  with:
    branch: develop
    output: develop-stats
```

### Custom Config File

```yaml
- uses: shenxianpeng/gitstats-action@v1
  with:
    config_file: .gitstats
    output: custom-report
```

### Deploy Report to GitHub Pages

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

## What's Included in the Report

The generated report includes:

- **General statistics**: total files, lines of code, commits, authors
- **Activity timeline**: commits by hour/day/month/year
- **Authors breakdown**: commits per author, lines added/removed
- **File statistics**: lines of code per file, file types distribution
- **Tags analysis**: version tags and their statistics

## How It Works

This is a **composite action** — it runs directly on the runner without Docker overhead.
On first use, it installs the [gitstats](https://pypi.org/project/gitstats/) Python package.

## Requirements

- `actions/checkout@v4` with `fetch-depth: 0` **must** be run before this action to fetch full git history
- Runner: `ubuntu-latest`

## License

MIT © Xianpeng Shen
