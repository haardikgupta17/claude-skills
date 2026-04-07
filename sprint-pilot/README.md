# sprint-pilot

AI-powered sprint co-pilot for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Fetches your Jira sprint tickets, deep-analyzes each one with parallel AI subagents, tracks velocity across sprints, detects scope creep, and produces copy-paste sprint reviews — all from your terminal.

## What it does

- Fetches your open sprint tickets from Jira
- Spawns parallel AI subagents to deep-analyze each ticket (searches Confluence for docs too)
- Generates developer-ready action briefs: what to do, where to start, who to talk to
- Flags carry-overs from previous sprints and detects scope additions
- Produces a color-coded Excel tracker with statuses, priorities, and AI-generated briefs
- Tracks velocity across sprints with resolution rate, scope creep, and carry-over trends
- Generates a 4-panel matplotlib velocity dashboard
- Creates copy-paste sprint reviews for Teams/Slack

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with Jira MCP server configured
- Python 3.8+
- Jira project with sprint boards

## Setup

```bash
# 1. Install Python dependencies
pip install -r requirements.txt

# 2. Copy the skill file to your Claude Code commands directory
cp sprint-pilot.md ~/.claude/commands/sprint-pilot.md

# 3. Copy the Python script to your preferred location
cp update_sprint_tasks.py ~/Documents/sprint-pilot/update_sprint_tasks.py
```

## Configuration

Edit the top of `~/.claude/commands/sprint-pilot.md` and update:

```
JIRA_PROJECT   = YOUR_PROJECT_KEY        # e.g. MYTEAM, ENG, PLATFORM
JIRA_BASE_URL  = https://your-jira.atlassian.net/browse
SCRIPT_PATH    = /path/to/update_sprint_tasks.py
EXCEL_PATH     = /path/to/Sprint_Tasks.xlsx
DASH_PNG_PATH  = /path/to/Sprint_Velocity_Dashboard.png
```

## Commands

| Command | Description |
|---------|-------------|
| `/sprint-pilot --liftoff` | Launch sprint — AI-powered ticket analysis + action briefs |
| `/sprint-pilot --land` | Sprint closeout + velocity update |
| `/sprint-pilot --earnings` | Copy-paste sprint review for Teams/Slack |
| `/sprint-pilot --dashboard` | Regenerate velocity charts |
| `/sprint-pilot --fleet` | Team-wide view (all assignees) |
| `/sprint-pilot --summon PROJ-123` | Add a ticket mid-sprint (flagged as scope addition) |
| `/sprint-pilot --factory PROJ` | Switch Jira project for this run |
| `/sprint-pilot --showroom` | Show all commands |

Flags combine: `/sprint-pilot --land --fleet --earnings`

## How it works

### Liftoff (`--liftoff`)

1. Queries Jira for your open sprint tickets
2. Spawns a parallel AI subagent for each ticket that:
   - Analyzes the technical work required
   - Checks for open PRs, blockers, and dependencies
   - Searches Confluence for relevant documentation
   - Generates a developer-ready action brief
3. Creates a color-coded Excel sheet with all tickets and AI briefs
4. Records sprint baseline in the Velocity sheet
5. Prints a formatted briefing in your terminal

### Landing (`--land`)

1. Fetches final ticket statuses
2. Updates the Excel with color-coded results
3. Detects scope additions (tickets added mid-sprint)
4. Calculates velocity metrics (resolution rate, scope creep, carry-overs)
5. Auto-regenerates the velocity dashboard

### Earnings (`--earnings`)

Generates a markdown sprint review ready to paste into Teams, Slack, or email — with ticket links, resolution stats, and velocity metrics.

## Excel Structure

Each sprint gets its own sheet with 9 columns:

| Column | Content |
|--------|---------|
| S.No | Sequential number |
| Ticket Number | Jira key |
| Summary | Ticket title |
| Priority | Color-coded (P1 red to P5 grey) |
| Status | Color-coded (green=done, red=blocked, blue=in progress) |
| Action Brief | AI-generated next steps |
| Carry-Over | Y/N flag |
| Risk / Dependency Flag | Your notes |
| Notes | Your notes |

Plus two special sheets:
- **Velocity** — sprint-over-sprint metrics with conditional formatting
- **Dashboard** — embedded 4-panel velocity chart

## Customization

### Adding custom statuses

Edit the `STATUS_COLORS` dict in `update_sprint_tasks.py`:

```python
STATUS_COLORS = {
    "Blocked":              "FF0000",
    "In Progress":          "4472C4",
    "Done":                 "00B050",
    "Your Custom Status":   "9B59B6",  # add yours here
}
```

### Confluence integration

The AI subagents automatically search Confluence for relevant docs during ticket analysis. This requires a Confluence MCP server configured in your Claude Code setup. If you don't use Confluence, the skill works fine without it — just remove Step 2 from the subagent prompt in the skill file.
