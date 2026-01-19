# Team Tools Plugin

Internal productivity tools for Claude Code.

## Commands

| Command | Description |
|---------|-------------|
| `/team-tools:timesheet [period]` | Generate daily commit summary for timesheets |
| `/team-tools:standup` | Yesterday's work + today's plan for standups |
| `/team-tools:coverage-gaps [threshold]` | Find files with missing or low test coverage |

## Installation

**Slash command** (in Claude Code session):
```
/plugin install https://github.com/zero-copy-labs/zcl-claude-code-plugins.git
```

**CLI** (in terminal):
```bash
claude plugin install https://github.com/zero-copy-labs/zcl-claude-code-plugins.git
```

Optional `--scope` flag: `user` (default), `project`, or `local`.

## Command Details

### /team-tools:timesheet

Generates a copy-paste ready timesheet from your git history.

```
/team-tools:timesheet            # Today's commits
/team-tools:timesheet yesterday  # Yesterday's commits
/team-tools:timesheet 7          # Last 7 days
```

**Features:**
- Groups commits by day
- Extracts ticket IDs (JIRA-123, #456)
- Multiple export formats (text, markdown, CSV)

### /team-tools:standup

Generates a daily standup report.

```
/team-tools:standup
```

**Features:**
- Yesterday's commits and merged PRs
- Today's open PRs and pending reviews
- Smart weekend handling (Monday shows Friday)
- Prompts for blockers

**Requires:** `gh` CLI for PR integration (optional)

### /team-tools:coverage-gaps

Finds files lacking test coverage.

```
/team-tools:coverage-gaps      # 50% threshold (default)
/team-tools:coverage-gaps 80   # 80% threshold
```

**Features:**
- Auto-detects test framework (Jest, Vitest, Pytest, Go, etc.)
- Parses existing coverage reports (lcov, cobertura)
- Falls back to file naming convention analysis
- Prioritizes by recency, size, and criticality

## Requirements

- Git repository
- `gh` CLI (optional, for PR features in `/standup`)
- Coverage report (optional, for `/coverage-gaps` accuracy)

## File Structure

```
.claude-plugin/
└── plugin.json      # Plugin manifest
commands/
├── timesheet.md     # /team-tools:timesheet command
├── standup.md       # /team-tools:standup command
└── coverage-gaps.md # /team-tools:coverage-gaps command
```

## License

Internal use only.
