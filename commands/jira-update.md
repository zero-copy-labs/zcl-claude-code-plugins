---
name: jira-update
description: Sync git commits with Jira tickets - fetch assigned issues, correlate commits, and post updates
mcp_servers:
  - atlassian
---

# Jira Update Generator

Sync your local git activity with Jira tickets. Fetches your assigned issues, correlates commits by ticket ID, and optionally posts progress updates back to Jira.

## Usage

`/jira-update` - Full sync: fetch tickets, correlate commits, offer to post updates
`/jira-update status` - Quick view of your assigned tickets and their status
`/jira-update post` - Post today's commit summaries as comments to relevant tickets

## Prerequisites

This command requires the **Atlassian MCP server** to be configured.

## Instructions

When invoked, follow these steps:

### Step 0: Check for Jira MCP Server

First, verify the Atlassian MCP server is available by attempting to list available MCP tools. Check if any `mcp__atlassian__*` or `mcp__jira__*` tools are accessible.

**If NO Jira MCP is detected**, stop and display:

```
⚠️  Jira MCP Server Not Found

This command requires the Atlassian MCP server to connect to Jira.

To install, run:

  claude mcp add atlassian -- npx -y @anthropic/mcp-atlassian

Then configure your credentials:

  claude config set env.ATLASSIAN_URL "https://your-domain.atlassian.net"
  claude config set env.ATLASSIAN_EMAIL "your-email@company.com"
  claude config set env.ATLASSIAN_API_TOKEN "your-api-token"

To generate an API token:
  1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
  2. Click "Create API token"
  3. Copy the token and set it above

After setup, run /jira-update again.
```

**If MCP is detected**, proceed to Step 1.

### Step 1: Get Git User Info

```bash
git config user.email
git config user.name
```

### Step 2: Fetch Assigned Jira Tickets

Use the Atlassian MCP to get the user's assigned tickets:

```
mcp_atlassian.jira_search(jql="assignee = currentUser() AND status NOT IN (Done, Closed) ORDER BY updated DESC")
```

Store the returned tickets with their:
- Issue key (e.g., `PROJ-123`)
- Summary/title
- Status
- Priority
- Last updated timestamp

### Step 3: Extract Recent Commits

Get commits from the last 24 hours (or since last working day if Monday):

```bash
git log --author="$(git config user.email)" --since="1 day ago" --format="%H|%s|%ad" --date=short --no-merges
```

For Monday, expand the range:
```bash
git log --author="$(git config user.email)" --since="3 days ago" --format="%H|%s|%ad" --date=short --no-merges
```

### Step 4: Correlate Commits with Tickets

Parse commit messages to extract ticket references:
- Look for patterns: `PROJ-123`, `[PROJ-123]`, `(PROJ-123)`, or `#123`
- Match against the fetched Jira tickets from Step 2
- Group commits by their associated ticket

### Step 5: Generate Sync Report

Present a formatted report:

```
JIRA SYNC REPORT - [Date]

═══ Your Active Tickets ═══

[PROJ-123] Feature: User Authentication
  Status: In Progress | Priority: High
  Recent Commits (3):
    • abc1234: Implemented OAuth flow
    • def5678: Added token refresh logic
    • ghi9012: Fixed redirect URI handling

[PROJ-456] Bug: Login timeout issue
  Status: In Review | Priority: Medium
  Recent Commits (1):
    • jkl3456: Increased session timeout to 30min

═══ Commits Without Tickets ═══
  • mno7890: Updated README
  • pqr1234: Fixed typo in config

═══ Tickets Without Recent Activity ═══
  [PROJ-789] Refactor payment module - No commits in 3 days
```

### Step 6: Offer Update Actions

After displaying the report, ask:

> "Would you like me to post progress updates to any of these tickets?"
>
> Options:
> 1. Post summaries to all tickets with commits
> 2. Select specific tickets to update
> 3. Skip posting

### Step 7: Post Updates to Jira (if requested)

For each selected ticket, compose and post a comment using the MCP:

```
mcp_atlassian.jira_add_comment(
  issue_key="PROJ-123",
  body="Development progress update:\n\n• Implemented OAuth flow\n• Added token refresh logic\n• Fixed redirect URI handling\n\n_Auto-generated from git commits_"
)
```

Format the comment with:
- Clear bullet points for each commit
- Commit hashes for reference (optional, based on preference)
- Footer noting it's auto-generated

### Step 8: Confirm Success

After posting, confirm:

```
✓ Posted updates to:
  • PROJ-123 (3 commits summarized)
  • PROJ-456 (1 commit summarized)

View in Jira:
  https://your-domain.atlassian.net/browse/PROJ-123
  https://your-domain.atlassian.net/browse/PROJ-456
```

## Subcommand: `status`

Quick view mode - skip commit correlation:

1. Fetch assigned tickets (Step 2)
2. Display in compact format:

```
YOUR JIRA TICKETS

In Progress:
  [PROJ-123] User Authentication (High) - Updated 2h ago
  [PROJ-456] Login timeout (Medium) - Updated 1d ago

In Review:
  [PROJ-789] Payment refactor (Low) - Updated 3d ago

Blocked:
  [PROJ-012] API integration (High) - Updated 5d ago ⚠️
```

## Subcommand: `post`

Direct posting mode - skip interactive prompts:

1. Gather commits (Step 3)
2. Correlate with tickets (Step 4)
3. Immediately post updates to all matched tickets
4. Report results

## Fallback Behavior

- **MCP not available**: Display the installation instructions from Step 0
- **Auth fails (401/403)**: Prompt user to check credentials:
  ```
  ⚠️  Jira authentication failed. Please verify your credentials:

    claude config set env.ATLASSIAN_EMAIL "your-email@company.com"
    claude config set env.ATLASSIAN_API_TOKEN "your-api-token"
  ```
- **No assigned tickets**: "No active tickets assigned to you. Check your Jira filters."
- **No commits found**: "No commits in the selected time range."
- **Ticket pattern not found**: Add commits to "Without Tickets" section
- **Post fails**: Report error, continue with remaining tickets

## Example Output

```
JIRA SYNC REPORT - Tuesday, Jan 14

═══ Your Active Tickets ═══

[AUTH-42] Implement SSO for enterprise clients
  Status: In Progress | Priority: High
  Recent Commits (2):
    • a1b2c3d: Added SAML assertion parser
    • e4f5g6h: Integrated with identity provider SDK

[AUTH-38] Fix session persistence bug
  Status: Code Review | Priority: Critical
  Recent Commits (1):
    • i7j8k9l: Store session in Redis instead of memory

═══ Commits Without Tickets ═══
  • m0n1o2p: Cleaned up unused imports

═══ Tickets Without Recent Activity ═══
  [AUTH-35] Documentation update - No commits in 5 days

───────────────────────────────────
Would you like me to post progress updates to any of these tickets?
```

