---
name: timesheet
description: Generate a daily timesheet entry from your git commit history, consolidating work by ticket.
---

# Timesheet Generator

Generate a consolidated, copy-paste ready timesheet entry for your daily time tracking.

## Usage

`/timesheet [period]`

- **No argument**: Generates timesheet for **Today** (since midnight).
- `yesterday`: Generates timesheet for **Yesterday** (entire 24h period).
- `[number]`: Generates timesheet for the last N days (e.g. `/timesheet 7`).

## Instructions

When invoked, follow these steps:

### Step 1: Determine Date Range

- **Default (Today)**: `since="midnight"`
- **Yesterday**: `since="yesterday 00:00"` and `until="today 00:00"`
- **N days**: `since="{N} days ago"`

### Step 2: Get Git User Info

Get the current git user's email:
```bash
git config user.email
```

### Step 3: Extract Commits

Run this command (adjusting `since`/`until` based on Step 1):

```bash
git log --author="$(git config user.email)" --since="{since}" --until="{until}" --format="%s" --no-merges
```

### Step 4: Process and Consolidate

1. **Extract Ticket IDs**: Look for patterns like `PROJ-123`, `FEAT-456`, or `#[0-9]+` at the start of messages.
2. **Group by Ticket**: Consolidate multiple commits for the same ticket.
3. **Summarize**:
   - If a ticket has multiple commits, combine them into a coherent summary.
   - Example rule: "Implemented auth flow, Fixed login bug" -> "Implemented auth flow & fixed login bug".
   - If commits are repetitive (e.g., "wip", "fix"), ignore the noise and focus on the meaningful parts.

### Step 5: Format Output

Present the timesheet in this format, optimized for pasting into time tracking software:

```
TIMESHEET - [Date or Range]

[Ticket ID]
[Consolidated Summary]

[Ticket ID]
[Consolidated Summary]

---
Miscellaneous / General:
- [Commit message without ticket ID]
```

## Example Output

```
TIMESHEET - Tuesday, Jan 14

PROJ-123
Implemented user authentication flow, added unit tests, and fixed login redirect bug.

PROJ-456
Code review feedback addressed: updated variable names and extracted constants.

---
Miscellaneous:
- Updated README.md
- Tweaked build script
```
