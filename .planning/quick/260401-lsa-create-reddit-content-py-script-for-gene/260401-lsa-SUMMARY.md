---
phase: quick
plan: 260401-lsa
subsystem: pipeline
tags: [reddit, traffic, content-repurposing, cli]
dependency_graph:
  requires: [data/content.db, database.py, config.yaml]
  provides: [reddit_content.py, data/reddit/]
  affects: []
tech_stack:
  added: []
  patterns: [subprocess-claude-cli, sqlite-tracking-table, dated-markdown-output]
key_files:
  created:
    - /Users/toxic/Documents/Projects/affiliate-pipeline/reddit_content.py
  modified: []
key_decisions:
  - data/reddit/ output gitignored (intentional — local artifact for manual posting)
  - Timeout on single article gracefully skipped, others continue
  - UNIQUE(article_id) constraint prevents re-generation across runs
metrics:
  duration: "6 minutes"
  completed: "2026-04-01"
  tasks_completed: 2
  tasks_total: 2
  files_created: 1
  files_modified: 0
---

# Quick Task 260401-lsa: Reddit Content Generator Summary

**One-liner:** Reddit comment generator using Claude CLI subprocess to produce 150-300 word helpful comments with natural article links from published aquapicked.com articles.

## What Was Built

`reddit_content.py` — a standalone script in the pipeline root that:

1. Queries `data/content.db` for published articles (site='aquarium') not yet in the `reddit_comments` tracking table
2. Calls `claude -p <prompt>` via subprocess for each article, requesting a JSON response with `comment_text`, `suggested_subreddit`, and `article_url`
3. Parses JSON from Claude output using regex extraction (handles surrounding prose)
4. Writes all generated comments to `data/reddit/reddit-comments-YYYY-MM-DD.md`
5. Records processed article IDs in the `reddit_comments` table (UNIQUE constraint prevents re-generation)

### Category-to-Subreddit Mapping

| Category | Subreddit |
|---|---|
| Lighting, CO2 Systems, Substrates, Aquascaping | r/PlantedTank |
| Filters, Tanks, Heaters, Pumps, Test Kits, Fish Food | r/Aquariums |

## Task Results

### Task 1: Create reddit_content.py
- File created at `/Users/toxic/Documents/Projects/affiliate-pipeline/reddit_content.py`
- Imports cleanly, all required functionality implemented
- Commit: `ae411e5`

### Task 2: Test with dry-run and verify output
- Script ran successfully against live `data/content.db`
- First run: 4/5 comments generated (1 timed out — handled gracefully, skipped with warning)
- Output file created: `data/reddit/reddit-comments-2026-04-01.md`
- `reddit_comments` table populated with 4 entries
- Second run: 5 new articles processed (previously processed articles excluded — dedup confirmed)
- Comments are genuinely helpful and conversational, natural link placement

## Deviations from Plan

### Auto-fixed Issues

None.

### Notes

- `data/reddit/` output is gitignored (consistent with all `data/` content). This is intentional — output files are local artifacts for manual posting, not repo content.
- The `python` command is `python3` on this system. The plan's verify step used `python`; verified with `python3` instead. Script shebang uses `#!/usr/bin/env python3` for correctness.

## Self-Check

- `/Users/toxic/Documents/Projects/affiliate-pipeline/reddit_content.py` — FOUND
- `/Users/toxic/Documents/Projects/affiliate-pipeline/data/reddit/reddit-comments-2026-04-01.md` — EXISTS (gitignored, local artifact)
- Commit `ae411e5` — FOUND
- `reddit_comments` table in `data/content.db` — 9 rows confirmed across two runs
