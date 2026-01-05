# Team Tools Plugin

Internal productivity tools for Claude Code.

## Commands

| Command | Description |
|---------|-------------|
| `/team-tools:timesheet [days]` | Generate weekly commit summary for timesheets |
| `/team-tools:standup` | Yesterday's work + today's plan for standups |
| `/team-tools:coverage-gaps [threshold]` | Find files with missing or low test coverage |

## Installation

```
/plugin marketplace add git@github.com:zero-copy-labs/zcl-claude-code-plugins.git
/plugin install team-tools@zcl-claude-code-plugins
```

Or with HTTPS:
```
/plugin marketplace add https://github.com/zero-copy-labs/zcl-claude-code-plugins.git
/plugin install team-tools@zcl-claude-code-plugins
```

## Command Details

### /timesheet

Generates a copy-paste ready timesheet from your git history.

```
/timesheet       # Last 7 days (default)
/timesheet 14    # Last 14 days
```

**Features:**
- Groups commits by day
- Extracts ticket IDs (JIRA-123, #456)
- Multiple export formats (text, markdown, CSV)

### /standup

Generates a daily standup report.

```
/standup
```

**Features:**
- Yesterday's commits and merged PRs
- Today's open PRs and pending reviews
- Smart weekend handling (Monday shows Friday)
- Prompts for blockers

**Requires:** `gh` CLI for PR integration (optional)

### /coverage-gaps

Finds files lacking test coverage.

```
/coverage-gaps      # 50% threshold (default)
/coverage-gaps 80   # 80% threshold
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
├── plugin.json      # Plugin manifest
commands/
├── timesheet.md     # /timesheet command
├── standup.md       # /standup command
└── coverage-gaps.md # /coverage-gaps command
```

## License

Internal use only.
