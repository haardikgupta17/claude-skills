# Skill: /sprint-pilot

Your AI-powered sprint co-pilot. Fetches Jira data, deep-analyzes every ticket with ULTRATHINK subagents, tracks velocity, detects scope creep, and produces copy-paste sprint reviews.

## Configuration

```
JIRA_PROJECT   = PROJ-CODE
SCRIPT_PATH    = ~/sprint-tracker/update_sprint_tasks.py
EXCEL_PATH     = ~/sprint-tracker/Sprint_Tasks.xlsx
DASH_PNG_PATH  = ~/sprint-tracker/Sprint_Velocity_Dashboard.png
JIRA_COMMENT   = true                        (post developer brief as Jira comment; use --no-comment to skip)

# Repo paths — list all repos the subagents should search for relevant code.
# The agent will search ALL listed repos and identify the right one(s) per ticket.
# Use labels to help the agent understand what each repo contains.
REPOS:
  - path: ~/repos/my-project           label: "Main application code — DAGs, pipelines, ETL jobs"
  - path: ~/repos/my-datamart          label: "Data warehouse / datamart — SQL models, views, stored procs"
  - path: ~/repos/my-framework         label: "Shared data processing framework — used when no custom code exists"
```

---

## Commands

```
/sprint-pilot --liftoff       Launch sprint — deep AI analysis + action briefs
/sprint-pilot --land          Sprint closeout + velocity update
/sprint-pilot --earnings      Copy-paste sprint review for Teams/Slack
/sprint-pilot --dashboard     Regenerate velocity charts
/sprint-pilot --fleet         Team-wide view (all assignees)
/sprint-pilot --summon PROJ-123  Summon a ticket into the sprint mid-flight
/sprint-pilot --factory OTHER-PROJ  Switch Jira project
/sprint-pilot --no-comment    Skip posting developer briefs to Jira
/sprint-pilot --showroom      Show all commands
```

Flags combine: `/sprint-pilot --land --fleet --earnings`

### Mode detection
- `--showroom` → **SHOWROOM flow** (print commands overview, no Jira calls)
- `--liftoff` → **LIFTOFF flow** (required — no default)
- `--land` → **LAND flow**
- `--earnings` → **EARNINGS flow** (can combine with --land)
- `--dashboard` → **DASHBOARD flow**
- `--no-comment` → **Skip Jira commenting** (can combine with any mode)
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
 ║  --summon PROJ-123  Summon a ticket mid-sprint            ║
 ║  --earnings       Copy-paste sprint review              ║
 ║  --dashboard      Regenerate velocity charts            ║
 ║  --fleet          Team-wide view                        ║
 ║  --factory X      Switch Jira project                   ║
 ║  --no-comment     Skip Jira comment posting             ║
 ║  --showroom       You are here                          ║
 ╠══════════════════════════════════════════════════════════╣
 ║  Combine: --land --fleet --earnings                     ║
 ╚══════════════════════════════════════════════════════════╝

What happens under the hood:
- Fetches your open sprint tickets from Jira
- Spawns ULTRATHINK AI subagents to deep-analyze each ticket
- Searches your local codebase(s) to identify exactly which files need changes (if REPOS configured)
- Generates developer-ready briefs: what to do, where to start, which files to modify
- Posts detailed developer briefs as Jira comments (skip with --no-comment)
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

Each subagent must return THREE things in a structured format:
1. **ACTION_BRIEF** — a 3-5 line developer-ready summary (goes into Excel Action Brief column)
2. **DEV_CONTEXT** — a detailed developer-ready explanation (printed in terminal)
3. **FILES_TO_MODIFY** — list of specific files across configured repos that need changes (if REPOS configured)

**Subagent prompt template:**
```
You are a Staff-level Data Engineer / Senior Developer reviewing sprint tickets. Your job is to produce a brief so clear and specific that a developer can read it and START CODING IMMEDIATELY — zero ambiguity, zero guesswork.

ULTRATHINK deeply about this Jira ticket. Cross-reference it with the actual codebase, Confluence docs, and comments. Your output should read like a senior engineer sitting next to the developer saying "here's exactly what you need to do, here's where, and here's what to watch out for."

Ticket: {key}
Summary: {summary}
Status: {status}
Priority: {priority}
Description: {description}
Comments (latest first): {comments}

## Step 1: Analyze the ticket like a Staff Engineer
Think deeply about:
- What is the EXACT technical work required? Not vague — specific operations, transformations, or code changes.
- What's the current state — what's been done, what remains? Parse the comments for progress signals.
- Are there PRs open? What state are they in? Who needs to review/approve?
- Who was last involved and what did they say? Any unresolved discussions?
- Are there blockers, dependencies, or external teams the developer MUST coordinate with before starting?
- What files, systems, pipelines, tables, or APIs does this touch?
- Is there a deadline or urgency signal?
- What are the RISKS? What could go wrong? What should the developer test before deploying?
- Are there upstream/downstream dependencies that could break if this change is done incorrectly?
- If the ticket description is vague, what SPECIFIC questions should the developer ask to unblock themselves?

## Step 2: Search the local codebase(s) for relevant files

The user has configured the following repos:
{REPOS}

Search ALL configured repos to find the exact files that need to be modified for this ticket. This is critical — the developer should know EXACTLY where to start coding and in WHICH repo.

Strategy:
1. **Identify the right repo(s)** — Use the repo labels to narrow down. A ticket about a DAG → search the DAGs/pipelines repo. A ticket about a datamart table → search the datamart repo. A data processing error → also check the framework repo to understand the processing logic.

2. **Search each relevant repo** using Glob and Grep:
   - Glob for files matching keywords from the ticket (DAG names, table names, pipeline names, module names)
     - e.g. ticket mentions "fleet_csv_export" → Glob for `**/fleet_csv*` in each repo
   - Grep for specific identifiers across file contents:
     - Table names, column names, function names mentioned in the ticket
     - Error messages or log strings from the ticket description
     - Class names, DAG IDs, pipeline names, SQL table references

3. **Read the most relevant files** (top 3-5) to understand the current implementation and confirm they're the right files to modify.

4. **For each file you identify, note:**
   - The repo name and file path (relative to repo root), e.g. `servicedatamart/models/fleet/fleet_events.sql`
   - Which line/section needs changes
   - What kind of change is needed (add, modify, delete)
   - A brief description of the change

5. **Cross-repo awareness** — Some tickets require changes in multiple repos (e.g. add a column in the datamart SQL AND update the DAG that loads it). Identify ALL repos that need changes.

If no REPOS are configured, skip this step entirely.

## Step 3: Search Confluence for relevant documentation
Use the mcp__confluence__ConfluenceQuery or mcp__confluence__ConfluenceSearchPages tool to search for any relevant documentation, design specs, runbooks, or reference material related to this ticket. Search using key terms from the ticket summary and description (e.g. table names, DAG names, pipeline names, system names).

If you find relevant Confluence pages, include their titles and URLs in your response. If nothing relevant is found, skip this — don't mention it.

## Step 4: Return your response

Return your response in EXACTLY this format. Be SPECIFIC and PRESCRIPTIVE — the developer should be able to read this and start coding within 5 minutes:

FILES_TO_MODIFY:
- `[airflow-dags] dags/fleet/fleet_csv_export.py` (line 45) — Add `quoting=csv.QUOTE_ALL` to `to_csv()` in `export_fleet_data()`
- `[servicedatamart] models/fleet/fleet_events.sql` (line 120-135) — Add `vin_region` column to SELECT
- `[airflow-dags] tests/test_fleet_csv.py` — Add test case: CSV with commas in VIN → verify escaping
(If no REPOS are configured, write "N/A — no repos configured")

ACTION_BRIEF: <3-5 line PRESCRIPTIVE summary. Not "investigate X" — instead "modify function Y in file Z to do W". Cover: (1) the exact change needed, (2) which files to touch, (3) dependencies/risks to watch for, (4) what "done" looks like including how to verify. This goes into both the Excel sheet and the Jira comment. The developer should read this and know EXACTLY what to do without opening Jira or Confluence.>

DEV_CONTEXT: <A thorough senior-developer-level explanation. As detailed as needed (typically 6-12 sentences). Structure it as:
1. WHAT: The exact technical change — functions to modify, SQL to update, configs to change
2. WHERE: File paths with line numbers, which functions/classes, which DAG/pipeline
3. HOW: Step-by-step approach — "First do X, then Y, then Z"
4. DEPENDENCIES: Other teams, systems, or tickets that must be coordinated with
5. RISKS: What could break, what to test, edge cases to handle, rollback plan
6. VERIFY: How to confirm the change works — specific test commands, expected outputs, logs to check
7. REFERENCES: Confluence docs, related PRs, Slack threads (if found)
If the ticket lacks sufficient detail, end with "⚠ Clarify before starting:" followed by 2-3 SPECIFIC questions the developer should ask the requestor.>
```

Spawn all subagents in parallel. Collect their responses. Parse FILES_TO_MODIFY, ACTION_BRIEF, and DEV_CONTEXT from each.

### Step 3: Build JSON and run script

Build JSON array with `action_brief` (the short version) populated from subagent responses:
```json
[
  {
    "key": "PROJ-12345",
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

### Step 4: Post developer briefs to Jira (unless --no-comment)

If `JIRA_COMMENT` is `true` AND `--no-comment` flag is NOT set, post a formatted comment on each Jira ticket using `mcp__jira__AddJiraComment`.

**Comment format (Jira wiki markup):**
```
h3. 🤖 Sprint Pilot — Developer Brief

h4. 📁 Files to Modify
{noformat}
• path/to/file1.py (line 45) — Add quoting=csv.QUOTE_ALL to to_csv() in export_fleet_data()
• path/to/file2.py (line 120-135) — Replace SELECT query to include vin_region column
• tests/test_file1.py — Add test: CSV with commas in VIN → verify escaping
{noformat}

h4. 📋 What To Do
<ACTION_BRIEF — prescriptive, 3-5 lines: exact change, files, risks, done criteria>

h4. 📖 Full Developer Context
<DEV_CONTEXT — structured as WHAT / WHERE / HOW / DEPENDENCIES / RISKS / VERIFY / REFERENCES>

h4. ⚠ Dependencies & Risks
• <Upstream/downstream dependencies>
• <What could break, edge cases>
• <Teams to coordinate with>

h4. 🔗 References
• Confluence: <links if found>
• Related PRs: <if mentioned>

----
_Auto-generated by Sprint Pilot during sprint liftoff. Not a human comment._
```

**IMPORTANT: Comments must be restricted to "Developers" role** so only the dev team sees the AI-generated briefs — not customers, PMs, or external stakeholders.

Since `mcp__jira__AddJiraComment` does not support visibility restrictions, use the Jira REST API directly via Bash:

```bash
curl -s -X POST \
  -H "Authorization: Bearer $(mcp__jira token or use stored credentials)" \
  -H "Content-Type: application/json" \
  "https://<jira-instance>/rest/api/2/issue/{ticket_key}/comment" \
  -d '{
    "body": "<comment text in Jira wiki markup>",
    "visibility": {
      "type": "role",
      "value": "Developers"
    }
  }'
```

Alternatively, if using `mcp__jira__AddJiraComment`, append the visibility restriction in the input JSON if the tool supports extra fields. Test with one ticket first to confirm the restriction works — check the comment shows the 🔒 lock icon and "Restricted to Developers" label in Jira.

Post all comments in parallel. This ensures the developer sees the brief the moment they open their ticket in Jira — no context switching needed.

If no `REPOS` are configured, omit the "Files to Modify" section from the comment.

### Step 5: Print the liftoff briefing

Print the launch header, then a per-ticket deep brief:

```
 🚀 T-minus 0 ... LIFTOFF
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Sprint: <sprint_name>
 Mission: X tickets | Y carry-overs | Z fresh
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### ⚠ Carry-Overs

**1. PROJ-12345 — Fix CSV quoting in MSSQL pipeline** | 🟠 P2 | Dev Assigned
> 📁 `dags/mssql_processor.py` (line 45) · `tests/test_mssql.py`
>
> PR #5608 open — needs review + merge
>
> The PR modifies `mssql_processor.py` to add `quoting=QUOTE_ALL` to the `to_csv()` call,
> fixing the escaping bug that was corrupting rows with commas in the VIN field.
> Sumanth reviewed on Mar 28 with 2 minor comments on error handling in the retry logic.
> Address those comments, push the fix, and get a re-approval. Once merged, the
> `fleet_csv_export` DAG will pick it up on the next scheduled run (daily 02:00 UTC).
> Verify by checking the output CSV in S3 for the next day's partition.

---

### Fresh Tickets

**2. PROJ-67890 — Set up CDC for fleet_events table** | 🟡 P3 | New
> 📁 No existing files found — greenfield implementation
>
> Define CDC access requirements and submit DBA request
>
> This ticket requires enabling Change Data Capture on the `fleet_events` table in the
> Service EDW so downstream analytics can consume incremental changes instead of full loads.
> The description mentions "CDC access" but doesn't specify which columns need tracking
> or what the target sink is (Kafka? S3? Databricks?).
> Questions to clarify:
> - Which specific columns on `fleet_events` need change tracking?
> - What's the target destination — Kafka topic, S3 path, or direct Databricks ingestion?
> - Is there an existing CDC pattern in this project to follow, or is this greenfield?

---

Sprint: <name> | Total: X | Carried over: Y | Fresh: Z
Action Brief column populated in Excel | {EXCEL_PATH}
```

Key formatting rules:
- Carried-over tickets grouped first under `### ⚠ Carry-Overs`
- Fresh tickets grouped second under `### Fresh Tickets`
- Each ticket shows: **#. KEY — Summary** | priority emoji | status
- First quote line = 📁 FILES_TO_MODIFY (if repo configured)
- Next quote line = ACTION_BRIEF (bold/concise)
- Remaining quote lines = DEV_CONTEXT (the developer explanation)
- Priority indicators: P1=🔴 P2=🟠 P3=🟡 P4=🟢 P5=⚪
- Separator `---` between tickets

### Step 6: User confirmation

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
 - PROJ-12345: Summary
 - PROJ-12345: Summary

 ❌ Blocked (Y)
 - PROJ-12345: Summary — Blocker: <reason>

 ⚠ Carrying Over (Z)
 - PROJ-12345: Summary

 📌 Scope Additions (W)
 - PROJ-12345: Summary — added mid-sprint

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
• [PROJ-12345](https://your-jira-instance.com/browse/PROJ-12345) — Summary
• ...

**In Progress** (carrying over)
• [PROJ-12345](link) — Summary | Status: Dev Assigned

**Blocked** (X tickets)
• [PROJ-12345](link) — Summary | Blocker: <reason>

**Unplanned Work** (scope additions: X tickets)
• [PROJ-12345](link) — Summary

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

## SUMMON Mode (--summon PROJ-12345)

Mid-sprint ticket addition. You're summoning a new ticket into an active sprint — it's automatically flagged as a scope addition.

Usage: `/sprint-pilot --summon PROJ-12345`
Also works with multiple: `/sprint-pilot --summon PROJ-12345 PROJ-12346`

### Flow:

1. **Fetch ticket(s)** via `mcp__jira__FetchJiraIssue` for each ticket key provided
2. **Extract sprint name** from `customfield_11501` on the ticket (parse `name=...` where `state=ACTIVE`)
3. **Spawn ULTRATHINK subagent** to analyze the ticket (same prompt as liftoff — searches codebase, searches Confluence, returns FILES_TO_MODIFY + ACTION_BRIEF + DEV_CONTEXT)
4. **Post Jira comment** with the developer brief (same format as liftoff Step 4, unless `--no-comment`)
5. **Run script:**
```bash
echo '<json>' | python3 "{SCRIPT_PATH}" --summon --sprint "<sprint_name>"
```
   The script:
   - Appends the ticket at the bottom of the sprint sheet in amber (scope addition)
   - Sets Carry-Over = N, Risk = "📌 Scope Addition"
   - Skips if the ticket is already in the sheet (prints "Skipped — already in sheet")

6. **Print the summon briefing:**
```
 ⚡ SUMMONED INTO SPRINT
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Sprint: <sprint_name>

 📌 PROJ-12345 — Summary | 🟠 P2 | Dev Assigned
 > <ACTION_BRIEF>
 >
 > <DEV_CONTEXT — full developer brief>

 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Added as scope addition | Total tickets now: X
```

7. **Post-Completion Flow** — ask user if everything looks good, offer to open Excel.

---

## FACTORY Override (--factory)

Replace `{JIRA_PROJECT}` with the specified project key for this run only.

---

## Excel Structure (9 columns per sprint sheet)

| Column | Header | Content |
|--------|--------|---------|
| A | S.No | Sequential number |
| B | Ticket Number | Jira key (PROJ-12345) |
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
- **Subagents search multiple repos** — when REPOS are configured, subagents search across all listed repositories using Grep/Glob/Read to find exactly which files need changes, with cross-repo awareness for multi-repo tickets
- **Jira comments posted in parallel** — after analysis, developer briefs are posted as comments on each ticket simultaneously using mcp__jira__AddJiraComment
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

## Sharing

1. Copy `update_sprint_tasks.py` to teammate's machine
2. Copy this skill file to their `~/.claude/commands/`
3. Update paths in Configuration section (especially `REPOS` to their local clones)
4. `pip install openpyxl matplotlib`
5. Run `/sprint-pilot --showroom` to see all commands
