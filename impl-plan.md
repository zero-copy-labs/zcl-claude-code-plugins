# Internal Team Plugin - Implementation Plan

## Overview
A private Claude Code plugin with three commands:
- `/timesheet` — Weekly commit summary for timesheets
- `/standup` — Yesterday's work + today's plan
- `/coverage-gaps` — Find untested files

---

## Feature 1: Timesheet (Day 1)

### Step A: Git Data Extraction
- [ ] Read git log for current repo
- [ ] Filter by author (current user's email)
- [ ] Filter by date range (default 7 days)

### Step B: Data Processing
- [ ] Group commits by day
- [ ] Extract ticket IDs (JIRA-123, #456)

### Step C: Output
- [ ] Day-by-day summary
- [ ] Copy-paste ready format

**Command:** `/timesheet [days]`

---

## Feature 2: Standup (Day 2-3)

### Step A: Gather Yesterday's Work
- [ ] Get commits from yesterday (or last working day if Monday)
- [ ] Get PRs merged yesterday
- [ ] Get PRs you reviewed yesterday

### Step B: Gather Today's Plan
- [ ] Get open PRs you authored (in progress)
- [ ] Get PRs awaiting your review
- [ ] Get current branch + uncommitted changes

### Step C: Format Standup
- [ ] "Yesterday I..." section from commits/PRs
- [ ] "Today I will..." section from open work
- [ ] "Blockers:" section (optional, prompt user)

**Command:** `/standup`

**Example Output:**
```
📅 Standup - Tuesday, Jan 7

Yesterday:
• Merged PR #42: Auth refactor
• Fixed login redirect bug (3 commits)
• Reviewed PR #38: API rate limiting

Today:
• Continue PR #45: User profile page (in progress)
• Review PR #50: Database migration

Blockers:
• None identified (add any?)
```

---

## Feature 3: Coverage Gaps (Day 4-5)

### Step A: Detect Test Framework
- [ ] Check for jest.config / vitest.config / pytest.ini / etc.
- [ ] Identify test file patterns (*_test.py, *.spec.ts, etc.)
- [ ] Find coverage report if exists (coverage/, .nyc_output/)

### Step B: Analyze Coverage
- [ ] Parse existing coverage report (lcov, cobertura, coverage.py)
- [ ] OR: Map source files to test files by naming convention
- [ ] Identify files with 0% or no corresponding test

### Step C: Report Gaps
- [ ] List files with no tests
- [ ] List files with low coverage (<50%)
- [ ] Prioritize by: file size, recent changes, complexity

**Command:** `/coverage-gaps [threshold]`

**Example Output:**
```
🔍 Coverage Gaps (threshold: 50%)

No tests found:
• src/utils/validation.ts (247 lines, modified 3 days ago)
• src/api/payments.ts (189 lines, modified today) ⚠️

Low coverage (<50%):
• src/auth/oauth.ts — 34% covered
• src/db/migrations.ts — 12% covered

Suggestion: Start with payments.ts (recent + no tests)
```

---

## File Structure

```
your-company/claude-plugins/
├── .claude-plugin/
│   ├── plugin.json            # Plugin manifest
│   └── marketplace.json       # Marketplace definition
├── commands/
│   ├── timesheet.md           # /timesheet command
│   ├── standup.md             # /standup command
│   └── coverage-gaps.md       # /coverage-gaps command
└── README.md
```

---

## Plugin Manifest

```json
{
  "name": "team-tools",
  "version": "1.0.0",
  "description": "Internal productivity tools",
  "author": {
    "name": "Your Team"
  },
  "components": {
    "commands": ["commands/"]
  }
}
```

---

## Rollout Plan

| Day | Task | Deliverable |
|-----|------|-------------|
| 1 | Build /timesheet | Working timesheet command |
| 2 | Build /standup (git part) | Yesterday's commits |
| 3 | Add PR integration to standup | Full standup output |
| 4 | Build /coverage-gaps detector | Framework detection |
| 5 | Coverage report parsing | Gap report |
| 6 | Testing + team feedback | Ready for team use |
| 7 | Docs + polish | Ship v1.0 |
