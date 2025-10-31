# Claude Code Custom Commands

  This directory contains custom slash commands for Claude Code that automate common development workflows and productivity tasks.

  ## Available Commands

  ### `/monthly-summary` - GitHub Contribution Tracker

  Automates the tedious process of tracking monthly accomplishments for performance reviews. Generates a comprehensive summary of GitHub contributions with stats,
  categorization, and business impact analysis.

  #### What It Does

  - Fetches all PRs authored and reviewed for a specified month
  - Categorizes contributions by project area and type (features, bugs, refactoring, etc.)
  - Extracts business impact and linked tickets from PR descriptions
  - Generates a polished markdown report with statistics
  - Uploads to a private GitHub repository for long-term tracking

  #### Prerequisites

  ```bash
  # Install GitHub CLI
  brew install gh
  gh auth login

  Setup

  1. Copy .claude/commands/monthly-summary.md to your repo's .claude/commands/ directory
  2. Restart Claude Code to load the new command

  Usage

  /monthly-summary                              # Current month, uploads to GitHub
  /monthly-summary 2025-09                      # Specific month
  /monthly-summary --username=yourname          # Different GitHub username
  /monthly-summary --local-only                 # Save locally only
  /monthly-summary 2025-09 --username=yourname  # Combine options

  Output

  - Local: .monthly-summary/YYYY-MM-monthly-summary.md
  - GitHub: Creates/updates private monthly-summaries repository
  - Includes: PR links, statistics, project areas, business impact, code review activity

  Roadmap

  Automation & Integrations:
  - Auto-schedule monthly runs with Slack notifications
  - Support multiple repositories
  - Pull data from Slack, Notion, Apple Notes, JIRA

  Smarter Analysis:
  - Intelligent PR description parsing for business impact
  - Auto-identify "Wins" using heuristics (LOC, reviewers, mentions)
  - Enhanced executive summary generation

  Trend Tracking:
  - Month-over-month comparison metrics
  - Year-end rollup aggregating all summaries
