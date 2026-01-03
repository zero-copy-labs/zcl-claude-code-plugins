---
name: timesheet
description: Generate a weekly commit summary for timesheets, grouped by day with ticket ID extraction
---

# Timesheet Generator

Generate a copy-paste ready timesheet from your git commit history.

## Usage

`/timesheet [days]` - Generate timesheet for the last N days (default: 7)

## Instructions

When invoked, follow these steps:

### Step 1: Get Git User Info

Run this command to get the current git user's email:

```bash
git config user.email
```

### Step 2: Extract Commits

Run this command to get commits for the specified date range (replace `{days}` with the argument or default to 7):

```bash
git log --author="$(git config user.email)" --since="{days} days ago" --format="%H|%ad|%s" --date=short
```

### Step 3: Process and Group

Parse the output and:
1. Group commits by date
2. Extract ticket IDs using these patterns:
   - JIRA-style: `[A-Z]+-[0-9]+` (e.g., PROJ-123, FEAT-456)
   - GitHub-style: `#[0-9]+` (e.g., #42, #100)
3. Summarize commit messages per day

### Step 4: Format Output

Present the timesheet in this format:

```
TIMESHEET - Week of [start date] to [end date]

[Day, Date]
• [Ticket ID]: [Summary of work]
• [Ticket ID]: [Summary of work]

[Day, Date]
• [Ticket ID]: [Summary of work]

---
Total commits: [count]
Days with activity: [count]
```

### Step 5: Offer Export Options

Ask the user if they want:
- Plain text (copy-paste ready)
- Markdown table format
- CSV format for spreadsheets

## Example Output

```
TIMESHEET - Week of Jan 1 to Jan 7

Monday, Jan 6
• PROJ-123: Implemented user authentication flow
• PROJ-124: Fixed login redirect issue

Tuesday, Jan 7
• #42: Added unit tests for auth module
• PROJ-125: Code review feedback addressed

---
Total commits: 8
Days with activity: 2
```

## Notes

- If no commits found, suggest checking the author email or date range
- Commits without ticket IDs are grouped under "General"
- Weekend commits are included but marked with (weekend) tag
