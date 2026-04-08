# /sprint-pilot — AI-Powered Sprint Co-Pilot

Your AI-powered sprint management skill for Claude Code. Fetches Jira data, deep-analyzes every ticket with Jarvis AI subagents (including Confluence research), tracks velocity across sprints, detects scope creep, and produces copy-paste sprint reviews.

## Commands

```
/sprint-pilot --liftoff          Launch sprint — AI-analyzed brief for every ticket
/sprint-pilot --land             Sprint closeout — final statuses + velocity update
/sprint-pilot --summon PROJ-123    Summon a ticket mid-sprint (auto-flagged as scope addition)
/sprint-pilot --earnings         Copy-paste sprint review message for Teams/Slack
/sprint-pilot --dashboard        Regenerate velocity charts
/sprint-pilot --fleet            Team-wide view (all assignees)
/sprint-pilot --factory OTHER-PROJ     Switch to a different Jira project
/sprint-pilot --no-comment       Skip posting developer briefs to Jira
/sprint-pilot --showroom         Show all commands
```

Flags combine: `/sprint-pilot --land --fleet --earnings`

## What it does

- Fetches your open sprint tickets from Jira
- Spawns Jarvis subagents (Sonnet, ~5x cheaper than Opus) to deep-analyze each ticket — reads description, comments, PRs, blockers, and searches Confluence for relevant docs
- Searches multiple local repos in priority order to identify exactly which files need changes, with line-level precision — stops searching once files are found to save tokens
- Posts detailed developer briefs as **restricted Jira comments** (Developers role only, via `curl` + `JIRA_PAT`) — developers see guidance the moment they open their ticket
- Generates developer-ready briefs: what to do, where to start, which files to modify, and smart clarifying questions for vague tickets
- Flags carry-overs from previous sprint (Y/N)
- Detects scope additions (tickets added mid-sprint)
- Updates Excel tracker with color-coded statuses, priorities, and action briefs
- Tracks velocity across sprints with resolution rate, scope creep, and carry-over trends
- Auto-generates a 4-panel velocity dashboard chart embedded in Excel

## Setup (2 minutes)

1. Copy `sprint-pilot.md` to `~/.claude/commands/`
2. Copy `update_sprint_tasks.py` to your working directory
3. Edit the paths at the top of `sprint-pilot.md`:
   ```
   JIRA_PROJECT   = <your Jira project key>
   SCRIPT_PATH    = <path to update_sprint_tasks.py>
   EXCEL_PATH     = <where you want Sprint_Tasks.xlsx>
   DASH_PNG_PATH  = <where you want the dashboard PNG>
   JIRA_COMMENT   = true                                  (posts developer briefs to Jira tickets)
   ```
4. Configure your repos in priority order (optional — enables codebase-aware analysis):
   ```
   REPOS:
     1. path: ~/repos/my-datamart       label: "Datamart SQL models (PRIMARY — searched first)"
     2. path: ~/repos/my-dags           label: "DAGs and pipeline code"
     3. path: ~/repos/my-framework      label: "Shared data processing framework"
   ```
5. Set up a Jira PAT for restricted comments (optional but recommended):
   ```bash
   # Create token at: https://<your-jira>/secure/ViewProfile.jspa → Personal Access Tokens
   echo 'export JIRA_PAT="your-token-here"' >> ~/.zshrc && source ~/.zshrc
   ```
6. Install dependencies:
   ```bash
   pip install openpyxl matplotlib
   ```
7. Run `/sprint-pilot --showroom` to verify

## Sample Excel

`Sprint_Sample.xlsx` is included — open it to see the format with 3 sprints of sample data, velocity tracking, and the embedded dashboard chart. No real data.

## Files

| File | Purpose |
|------|---------|
| `sprint-pilot.md` | Claude Code skill file (copy to `~/.claude/commands/`) |
| `update_sprint_tasks.py` | Sprint tracker engine (Excel generation, velocity tracking, dashboard) |
| `Sprint_Sample.xlsx` | Sample Excel with 3 fake sprints for demo |
| `Sprint_Sample_Dashboard.png` | Sample velocity dashboard chart |

## Excel Structure (9 columns per sprint)

| Column | Header | Content |
|--------|--------|---------|
| A | S.No | Sequential number |
| B | Ticket Number | Jira key |
| C | Summary | Ticket title |
| D | Priority | Color-coded (P1 red, P2 orange, P3 yellow, P4 green, P5 grey) |
| E | Status | Color-coded (green=done, red=blocked, blue=dev) |
| F | Action Brief | AI-generated developer brief (light blue background) |
| G | Carry-Over | Y/N flag |
| H | Risk / Dependency Flag | Your column — empty unless you fill it |
| I | Notes | Your column — never overwritten |

Special sheets: **Dashboard** (embedded chart), **Velocity** (sprint metrics)

## Requirements

- Claude Code with Jira MCP server configured
- Python 3.8+ with `openpyxl` and `matplotlib`
- Works with any Jira project that uses sprints
