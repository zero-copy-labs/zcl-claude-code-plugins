---
name: standup
description: Generate a daily standup report from yesterday's work and today's plan
---

# Standup Generator

Generate a ready-to-share standup report from your git activity and open work.

## Usage

`/standup` - Generate standup for today

## Instructions

When invoked, follow these steps:

### Step 1: Determine Yesterday

Calculate "yesterday" intelligently:
- If today is Monday, use Friday
- If today is Sunday, use Friday
- If today is Saturday, use Friday
- Otherwise, use the previous day

### Step 2: Gather Yesterday's Work

#### A. Get Yesterday's Commits

```bash
git log --author="$(git config user.email)" --since="yesterday 00:00" --until="today 00:00" --format="%s" --no-merges
```

For Monday standups, adjust to get Friday's commits:
```bash
git log --author="$(git config user.email)" --since="3 days ago 00:00" --until="2 days ago 00:00" --format="%s" --no-merges
```

#### B. Get Merged PRs (if gh CLI available)

```bash
gh pr list --author @me --state merged --search "merged:>=$(date -v-1d +%Y-%m-%d)" --json number,title --jq '.[] | "PR #\(.number): \(.title)"'
```

#### C. Get PRs You Reviewed

```bash
gh pr list --state all --search "reviewed-by:@me updated:>=$(date -v-1d +%Y-%m-%d)" --json number,title --jq '.[] | "Reviewed PR #\(.number): \(.title)"'
```

### Step 3: Gather Today's Plan

#### A. Get Open PRs You Authored

```bash
gh pr list --author @me --state open --json number,title,isDraft --jq '.[] | "PR #\(.number): \(.title)" + (if .isDraft then " (draft)" else "" end)'
```

#### B. Get PRs Awaiting Your Review

```bash
gh pr list --search "review-requested:@me" --state open --json number,title --jq '.[] | "Review PR #\(.number): \(.title)"'
```

#### C. Get Current Branch Status

```bash
git branch --show-current
git status --porcelain | wc -l | xargs echo "uncommitted changes:"
```

### Step 4: Format Standup

Present in this format:

```
STANDUP - [Day], [Date]

Yesterday:
• [Merged PR / Commit summary]
• [Reviewed PR]
• [Other work done]

Today:
• [Continue/Complete PR in progress]
• [Review pending PRs]
• [Current branch work]

Blockers:
• None identified
```

### Step 5: Ask About Blockers

After presenting the standup, ask:
> "Any blockers I should add to this standup?"

If the user provides blockers, update the Blockers section.

## Example Output

```
STANDUP - Tuesday, Jan 7

Yesterday:
• Merged PR #42: Auth refactor
• Fixed login redirect bug (3 commits)
• Reviewed PR #38: API rate limiting

Today:
• Continue PR #45: User profile page (in progress)
• Review PR #50: Database migration
• Working on feature/user-settings branch (5 uncommitted changes)

Blockers:
• None identified (add any?)
```

## Fallback Behavior

- If `gh` CLI is not available, skip PR-related steps and note: "Install gh CLI for PR integration"
- If no commits found, mention: "No commits yesterday - was this expected?"
- For Monday standups, expand to include Thursday if Friday was also empty
