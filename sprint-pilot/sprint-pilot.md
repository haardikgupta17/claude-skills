# Skill: /sprint-pilot

Your AI-powered sprint co-pilot for Claude Code. Fetches Jira data, deep-analyzes every ticket with ULTRATHINK subagents, tracks velocity, detects scope creep, and produces copy-paste sprint reviews.

## Configuration

Update these to match your environment:

```
JIRA_PROJECT   = YOUR_PROJECT_KEY
JIRA_BASE_URL  = https://your-jira-instance.atlassian.net/browse
SCRIPT_PATH    = /path/to/update_sprint_tasks.py
EXCEL_PATH     = /path/to/Sprint_Tasks.xlsx
DASH_PNG_PATH  = /path/to/Sprint_Velocity_Dashboard.png
```

---

## Commands

```
/sprint-pilot --liftoff        Launch sprint — deep AI analysis + action briefs
/sprint-pilot --land           Sprint closeout + velocity update
/sprint-pilot --earnings       Copy-paste sprint review for Teams/Slack
/sprint-pilot --dashboard      Regenerate velocity charts
/sprint-pilot --fleet          Team-wide view (all assignees)
/sprint-pilot --summon PROJ-X  Summon a ticket into the sprint mid-flight
/sprint-pilot --factory PROJ   Switch Jira project
/sprint-pilot --showroom       Show all commands
```

Flags combine: `/sprint-pilot --land --fleet --earnings`

### Mode detection
- `--showroom` → **SHOWROOM flow** (print commands overview, no Jira calls)
- `--liftoff` → **LIFTOFF flow** (required — no default)
- `--land` → **LAND flow**
- `--earnings` → **EARNINGS flow** (can combine with --land)
- `--dashboard` → **DASHBOARD flow**
- No flags → print showroom and ask user to pick a command

---

## SHOWROOM Mode (--showroom)

Print this formatted overview — no Jira calls, no script execution:

```
 ╔══════════════════════════════════════════════════════════╗
 ║           /sprint-pilot — Mission Control               ║
 ╠══════════════════════════════════════════════════════════╣
 ║  --liftoff        Launch sprint — AI action briefs      ║
 ║  --land           Sprint closeout + velocity            ║
 ║  --summon PROJ-X  Summon a ticket mid-sprint            ║
 ║  --earnings       Copy-paste sprint review              ║
 ║  --dashboard      Regenerate velocity charts            ║
 ║  --fleet          Team-wide view                        ║
 ║  --factory X      Switch Jira project                   ║
 ║  --showroom       You are here                          ║
 ╠══════════════════════════════════════════════════════════╣
 ║  Combine: --land --fleet --earnings                     ║
 ╚══════════════════════════════════════════════════════════╝

What happens under the hood:
- Fetches your open sprint tickets from Jira
- Spawns ULTRATHINK AI subagents to deep-analyze each ticket
- Generates developer-ready briefs: what to do, where to start, who to talk to
- Flags carry-overs (Y/N) and detects scope additions
- Updates Excel tracker with color-coded statuses, priorities, and action briefs
- Tracks velocity across sprints with resolution rate, scope creep, and carry-over trends

Excel: {EXCEL_PATH}
```

---

## LIFTOFF Mode (--liftoff)

**This is the signature feature.** When the user runs `/sprint-pilot --liftoff`, execute the full liftoff flow. If no flags are provided, print the `--showroom` output and ask the user to pick a command.

### Step 1: Fetch sprint tickets

Query via `mcp__jira__JiraQuery`:
```
project = {JIRA_PROJECT} AND sprint in openSprints() AND assignee = currentUser() AND statusCategory != Done ORDER BY priority ASC
```
Fields: `summary`, `priority`, `status`, `customfield_11501`, `description`, `comment`

Extract sprint name from `customfield_11501` on the first ticket (parse `name=...`).

### Step 2: ULTRATHINK multi-agent ticket analysis

For EACH ticket, spawn a **subagent** (using the Agent tool) to deeply analyze the ticket and produce a developer-ready brief. This parallelizes the analysis across all tickets.

Each subagent must return TWO things in a structured format:
1. **ACTION_BRIEF** — a 3-5 line developer-ready summary (goes into Excel Action Brief column)
2. **DEV_CONTEXT** — a detailed developer-ready explanation (printed in terminal)

**Subagent prompt template:**
```
You are a senior engineering lead doing sprint planning. ULTRATHINK about this Jira ticket and produce a developer-ready brief so the engineer knows exactly what to do, where to start, and who to talk to.

Ticket: {key}
Summary: {summary}
Status: {status}
Priority: {priority}
Description: {description}
Comments (latest first): {comments}

## Step 1: Analyze the ticket deeply
Think about:
- What is the actual technical work required?
- What's the current state — what's been done, what remains?
- Are there PRs open? What state are they in?
- Who was last involved and what did they say?
- Are there blockers, dependencies, or external teams involved?
- What files, systems, or pipelines does this touch?
- Is there a deadline or urgency signal?
- If the ticket description is vague or missing details, what questions should the developer ask the requestor to unblock themselves?

## Step 2: Search Confluence for relevant documentation
Use the mcp__confluence__ConfluenceQuery or mcp__confluence__ConfluenceSearchPages tool to search for any relevant documentation, design specs, runbooks, or reference material related to this ticket. Search using key terms from the ticket summary and description (e.g. table names, DAG names, pipeline names, system names).

If you find relevant Confluence pages, include their titles and URLs in your response. If nothing relevant is found, skip this — don't mention it.

## Step 3: Return your response

Return your response in EXACTLY this format:

ACTION_BRIEF: <3-5 line detailed summary covering: what the next step is, what's the current state, key people/PRs/systems involved, and what "done" looks like. This goes into the Excel sheet so the developer can read it and immediately understand the task without opening Jira. Include Confluence doc links if found.>
DEV_CONTEXT: <A thorough developer-ready explanation — as many sentences as needed (typically 4-8) to give the developer full context to get started. Be specific: mention file names, PR numbers, people to contact, systems involved, what's been tried, what remains, Confluence references, and what "done" looks like. If the ticket lacks sufficient detail, end with "Questions to clarify:" followed by 2-3 smart questions the developer should ask the requestor before starting.>
```

Spawn all subagents in parallel. Collect their responses. Parse ACTION_BRIEF and DEV_CONTEXT from each.

### Step 3: Build JSON and run script

Build JSON array with `action_brief` (the short version) populated from subagent responses:
```json
[
  {
    "key": "PROJ-XXXXX",
    "summary": "...",
    "priority": "P2-High",
    "status": "Dev Assigned",
    "action_brief": "PR #5608 open — needs review + merge"
  }
]
```

Run:
```bash
echo '<json>' | python3 "{SCRIPT_PATH}" --start --sprint "<sprint_name>"
```

### Step 4: Print the liftoff briefing

Print the launch header, then a per-ticket deep brief:

```
 🚀 T-minus 0 ... LIFTOFF
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Sprint: <sprint_name>
 Mission: X tickets | Y carry-overs | Z fresh
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### ⚠ Carry-Overs

**1. PROJ-XXXXX — Fix CSV quoting in MSSQL pipeline** | 🟠 P2 | Dev Assigned
> PR #5608 open — needs review + merge
>
> The PR modifies `mssql_processor.py` to add `quoting=QUOTE_ALL` to the `to_csv()` call,
> fixing the escaping bug that was corrupting rows with commas in the VIN field.
> Address review comments, push the fix, and get re-approval. Once merged, the
> DAG will pick it up on the next scheduled run. Verify by checking the output
> in the target destination for the next partition.

---

### Fresh Tickets

**2. PROJ-YYYYY — Set up CDC for events table** | 🟡 P3 | New
> Define CDC access requirements and submit DBA request
>
> This ticket requires enabling Change Data Capture on the target table so
> downstream analytics can consume incremental changes instead of full loads.
> The description mentions "CDC access" but doesn't specify which columns need tracking
> or what the target sink is (Kafka? S3? Databricks?).
> Questions to clarify:
> - Which specific columns need change tracking?
> - What's the target destination — Kafka topic, S3 path, or direct ingestion?
> - Is there an existing CDC pattern in this project to follow, or is this greenfield?

---

Sprint: <name> | Total: X | Carried over: Y | Fresh: Z
Action Brief column populated in Excel | {EXCEL_PATH}
```

Key formatting rules:
- Carried-over tickets grouped first under `### ⚠ Carry-Overs`
- Fresh tickets grouped second under `### Fresh Tickets`
- Each ticket shows: **#. KEY — Summary** | priority emoji | status
- First quote line = ACTION_BRIEF (bold/concise)
- Remaining quote lines = DEV_CONTEXT (the developer explanation)
- Priority indicators: P1=🔴 P2=🟠 P3=🟡 P4=🟢 P5=⚪
- Separator `---` between tickets

### Step 5: User confirmation

See **Post-Completion Flow** at the bottom of this file — applies to ALL modes.

---

## LAND Mode (--land)

### Step 1: Fetch ALL sprint tickets (no statusCategory filter)

```
project = {JIRA_PROJECT} AND sprint in openSprints() AND assignee = currentUser() ORDER BY priority ASC
```

### Step 2: Build JSON with final statuses, run script

```bash
echo '<json>' | python3 "{SCRIPT_PATH}" --end --sprint "<sprint_name>"
```

The script automatically:
- Updates status cells with color coding (green=done, red=blocked)
- Detects scope additions (tickets not in the baseline sheet)
- Appends scope additions in amber
- Preserves Action Brief and Notes columns from liftoff
- Updates Velocity sheet

### Step 3: Print landing report

```
 🛬 LANDING REPORT
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Mission: <sprint_name>
 Landing Status: <SUCCESS / PARTIAL SUCCESS / ROUGH LANDING>
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 ✅ Resolved (X)
 - PROJ-XXXXX: Summary
 - PROJ-XXXXX: Summary

 ❌ Blocked (Y)
 - PROJ-XXXXX: Summary — Blocker: <reason>

 ⚠ Carrying Over (Z)
 - PROJ-XXXXX: Summary

 📌 Scope Additions (W)
 - PROJ-XXXXX: Summary — added mid-sprint

 🔋 Sprint Fuel Gauge
 ████████████░░░░ XX% resolved
 ██░░░░░░░░░░░░░░ XX% scope creep
 ███░░░░░░░░░░░░░ XX% carry-over
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Resolved: X | Blocked: Y | Carrying: Z | Scope additions: W
```

Landing status logic:
- **SUCCESS** → 80%+ resolved, 0 blocked
- **PARTIAL SUCCESS** → 50-79% resolved OR any blocked
- **ROUGH LANDING** → <50% resolved

Fuel gauge: render using block chars (█ for filled, ░ for empty, 16 chars total).

---

## EARNINGS Mode (--earnings)

Run the same data collection as --land, then generate a **copy-paste-ready message**:

```
## Sprint Review: <sprint_name>
**Duration:** <start> — <end>  |  **Resolution:** X/Y (Z%)

**Completed** (X tickets)
• [PROJ-XXXXX]({JIRA_BASE_URL}/PROJ-XXXXX) — Summary
• ...

**In Progress** (carrying over)
• [PROJ-XXXXX](link) — Summary | Status: Dev Assigned

**Blocked** (X tickets)
• [PROJ-XXXXX](link) — Summary | Blocker: <reason>

**Unplanned Work** (scope additions: X tickets)
• [PROJ-XXXXX](link) — Summary

📊 Velocity: Z% resolution | W% scope creep | V carry-overs
```

---

## DASHBOARD Mode (--dashboard)

Regenerate the 4-panel matplotlib velocity dashboard:
1. Sprint Outcome Breakdown (stacked bar)
2. Resolution Rate Trend (line chart)
3. Planned vs Actual Scope (the scope creep visual)
4. Scope Creep Rate trend

Read data from the Velocity sheet in the Excel. Save PNG and embed in Dashboard sheet.
Requires: `matplotlib`

---

## FLEET Mode (--fleet)

Modify the Jira query to remove `assignee = currentUser()`:
```
project = {JIRA_PROJECT} AND sprint in openSprints() ORDER BY assignee ASC, priority ASC
```

Group output by assignee. Add "Assignee" as an extra info column in the printed output.

---

## SUMMON Mode (--summon PROJ-XXXXX)

Mid-sprint ticket addition. You're summoning a new ticket into an active sprint — it's automatically flagged as a scope addition.

Usage: `/sprint-pilot --summon PROJ-12345`
Also works with multiple: `/sprint-pilot --summon PROJ-12345 PROJ-12346`

### Flow:

1. **Fetch ticket(s)** via `mcp__jira__FetchJiraIssue` for each ticket key provided
2. **Extract sprint name** from `customfield_11501` on the ticket (parse `name=...` where `state=ACTIVE`)
3. **Spawn ULTRATHINK subagent** to analyze the ticket (same prompt as liftoff — searches Confluence for docs, returns ACTION_BRIEF + DEV_CONTEXT)
4. **Run script:**
```bash
echo '<json>' | python3 "{SCRIPT_PATH}" --summon --sprint "<sprint_name>"
```
   The script:
   - Appends the ticket at the bottom of the sprint sheet in amber (scope addition)
   - Sets Carry-Over = N, Risk = "📌 Scope Addition"
   - Skips if the ticket is already in the sheet (prints "Skipped — already in sheet")

5. **Print the summon briefing:**
```
 ⚡ SUMMONED INTO SPRINT
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Sprint: <sprint_name>

 📌 PROJ-XXXXX — Summary | 🟠 P2 | Dev Assigned
 > <ACTION_BRIEF>
 >
 > <DEV_CONTEXT — full developer brief>

 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Added as scope addition | Total tickets now: X
```

6. **Post-Completion Flow** — ask user if everything looks good, offer to open Excel.

---

## FACTORY Override (--factory)

Replace `{JIRA_PROJECT}` with the specified project key for this run only.

---

## Excel Structure (9 columns per sprint sheet)

| Column | Header | Content |
|--------|--------|---------|
| A | S.No | Sequential number |
| B | Ticket Number | Jira key (PROJ-XXXXX) |
| C | Summary | Ticket title |
| D | Priority | Color-coded (P1 red → P5 grey) |
| E | Status | Color-coded (green=done, red=blocked, blue=dev) |
| F | **Action Brief** | AI-generated next steps (light blue background) |
| G | Carry-Over | Y/N flag |
| H | Risk / Dependency Flag | User's column — empty unless specified |
| I | Notes | User's column — never overwritten |

Special sheets:
- **Dashboard** — embedded velocity chart image
- **Velocity** — sprint-over-sprint metrics with conditional formatting

---

## Optimization Notes

- **Subagents for ticket analysis** — each ticket is analyzed in parallel by a dedicated subagent, keeping the main context window clean and fast
- **Subagents CAN use MCP tools** — they search Confluence for relevant docs/runbooks autonomously during analysis
- **Batch the Jira query** — use one JiraQuery call in the main agent (not per-ticket), then pass data to subagents
- **JSON via stdin** — the script reads from stdin to avoid shell escaping issues with large payloads

## Post-Completion Flow (applies to ALL modes)

After completing ANY mode (--liftoff, --land, --earnings, --dashboard), ALWAYS:

1. Print the results/summary first
2. Then ask:
   ```
   Everything look good? Anything you'd like to adjust?
   Want me to open the Excel? (y/n)
   ```
3. If user says yes → run `open "{EXCEL_PATH}"`
4. If user wants changes → make them, then ask again
5. Do NOT auto-open the file without asking

---

## Setup

1. Install dependencies: `pip install openpyxl matplotlib`
2. Copy `update_sprint_tasks.py` to your preferred location
3. Copy `sprint-pilot.md` to `~/.claude/commands/`
4. Update the Configuration section at the top with your paths and Jira project key
5. Run `/sprint-pilot --showroom` to verify
