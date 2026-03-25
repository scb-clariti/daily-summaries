---
name: setup
description: Interactive setup wizard for Daily Summaries — configures identity, creates the private data repo, sets up scheduled tasks, and pushes everything to GitHub
---

## Daily Summaries — Setup Wizard

This skill walks you through a one-time setup. By the end you will have:
- A **private data repo** (your summaries and config live here, never public)
- **Scheduled tasks** that pull skills from this repo and write summaries to your data repo
- All MCP connectors tested and permissions granted

---

## STEP 0 — Detect environment

### OS & defaults
1. Run `uname -s` to detect OS:
   - `Darwin` → macOS
   - `MINGW*`, `MSYS*`, `CYGWIN*` → Windows (Git Bash)
2. Run `git config user.name` to get the default name.
3. Detect timezone:
   - macOS: `readlink /etc/localtime | sed 's|.*/zoneinfo/||'`
   - Windows: `powershell -Command "[System.TimeZoneInfo]::Local.Id"` then map to IANA (e.g., "Eastern Standard Time" → "America/Toronto", "Pacific Standard Time" → "America/Los_Angeles", "Central Standard Time" → "America/Chicago", "Mountain Standard Time" → "America/Denver")

### Detect SKILLS_REPO_PATH
4. Run `pwd` in Bash. This is `SKILLS_REPO_PATH` — the directory where this setup wizard is running (the cloned public repo).
5. Run `git rev-parse --is-inside-work-tree` to confirm we are inside the skills repo.

---

## STEP 1 — Existing config check

Check if a sibling data repo already exists by looking for directories named `*-journal` or `*-daily*` next to `SKILLS_REPO_PATH`. Also check if `config.yaml` exists anywhere nearby.

If a previous setup is detected, ask:
> "It looks like you may have run setup before. What would you like to do?"
> - Start fresh — create a new data repo and overwrite config
> - Re-run setup on existing data repo — update config only

If "Re-run on existing": ask for the `DATA_REPO_PATH` (the local path to the existing data repo), load `config.yaml` from there as defaults, and skip STEP 2 (data repo creation).

---

## STEP 2 — Identity

Ask:

**Full name**
> "What is your full name? This appears in summary file headers."
> Default: `git config user.name` or empty.

**Timezone**
> "What is your timezone?"
> Options: detected timezone (Recommended), America/Toronto, America/New_York, America/Chicago, America/Denver, America/Los_Angeles, America/Vancouver, Europe/London, Europe/Paris, Other
> Default: detected timezone.

**Slack user ID**
> "What is your Slack user ID? (Slack → click your profile photo → three dots → Copy member ID)"
> Default: empty.

---

## STEP 3 — Create the private data repo

The data repo stores your `config.yaml` and all generated summary files. It is always **private**.

Ask:
> "What should your private data repo be named?"
> Options:
> - `daily-summaries-journal` (Recommended)
> - Other (specify)

Ask:
> "Which GitHub account should it be created under?"
> Run `gh auth status` to detect the logged-in account. Offer that as default.

**Check if repo already exists:**
Run `gh repo list <account> --json name --jq '.[].name'` and check if the name is already taken.
- If it exists: ask "A repo named `<name>` already exists. Use it as your data repo?"
  - Yes: run `gh repo clone <account>/<name> <sibling_path>` to clone it locally (if not already cloned)
  - No: ask for a different name

**If creating new:**
1. Run: `gh repo create <account>/<name> --private --clone --clone-directory <sibling_path>`
   where `<sibling_path>` is the parent directory of `SKILLS_REPO_PATH` + the repo name.
   Example: if `SKILLS_REPO_PATH` = `/home/user/work/daily-summaries`, then `sibling_path` = `/home/user/work/daily-summaries-journal`
2. Verify the clone succeeded with `ls <sibling_path>`.

**If `gh` is not installed:**
Print:
```
GitHub CLI (gh) is required. Install it:
  macOS:   brew install gh
  Windows: winget install --id GitHub.cli
Then authenticate: gh auth login
```
Stop and ask the user to retry after installing.

Set `DATA_REPO_PATH` = the absolute local path to the data repo.

**Initialize data repo:**
- Create a `.gitignore` in `DATA_REPO_PATH`:
  ```
  # OS files
  .DS_Store
  Thumbs.db
  ```
- Create a `README.md` in `DATA_REPO_PATH`:
  ```markdown
  # Daily Summaries — Private Journal

  This is a private repository containing generated daily work summaries and configuration.
  It is managed automatically by Claude Code scheduled tasks.

  Do not edit files here manually unless you know what you're doing.
  Skills and setup instructions live in the companion public repo.
  ```
- Stage and commit these files: `cd DATA_REPO_PATH && git add . && git commit -m "Initialize data repo"`
- Push: `git push origin <branch>`

---

## STEP 4 — Google Drive for Desktop

Ask:
> "Do you have Google Drive for Desktop installed and syncing?"
> - Yes
> - No — skip Google Drive integration

If yes:
1. Auto-detect common paths:
   - macOS: `~/Library/CloudStorage/GoogleDrive-*/My Drive` or `~/Google Drive/My Drive`
   - Windows: `C:\Users\<username>\My Drive`, `G:\My Drive`, `C:\Users\<username>\Google Drive`
2. If found, propose it:
   > "Found Google Drive at: `<path>`. Is this correct?"
   > - Yes
   > - No, let me specify
3. Validate with `test -d`. If it doesn't exist, warn but continue.

If No: set `google_drive.enabled: false`.

---

## STEP 5 — Data sources

Ask (multi-select):
> "Which data sources should the daily summary pull from?"
> - Google Calendar (Recommended)
> - Gmail
> - Slack
> - Google Drive

For Gmail:
> "Should Gmail only include emails you acted on (replied, forwarded)?"
> - Yes, action-only (Recommended)
> - No, include all

For Slack:
> "Should Slack only include messages you sent?"
> - Yes, sent-only (Recommended)
> - No, include all

---

## STEP 6 — Summary format

Ask (multi-select):
> "Which sections should appear in your daily summary?"
> - Summary bullets (always on)
> - Decisions & Rationale
> - Open Loops
> - Blockers
> - Next Steps
> - Transcript Source (Recommended)

Default: all enabled.

Ask:
> "How many bullets should the summary aim for on a typical day?"
> - 5-10 (concise)
> - 5-15 (balanced, Recommended)
> - 10-20 (detailed)
> - Other

Ask:
> "Load yesterday's context to carry forward open loops and next steps?"
> - Yes (Recommended)
> - No

If yes:
> "How many prior days should it look back?"
> - 1 day — yesterday only (Recommended)
> - 2 days
> - 3 days

---

## STEP 7 — Schedule

Ask:
> "What time should the daily summary run?"
> - 4:00 PM
> - 5:00 PM (Recommended)
> - 6:00 PM
> - Other

Ask:
> "Which days should it run?"
> - Weekdays only — Monday to Friday (Recommended)
> - Every day — including weekends

Ask:
> "Enable weekly rollup? (synthesizes the week's daily summaries every Friday)"
> - Yes (Recommended)
> - No

---

## STEP 8 — Write config.yaml to DATA_REPO_PATH

Derive the file slug from the full name:
1. Lowercase
2. Remove accents: `echo "<name>" | iconv -f utf-8 -t ascii//TRANSLIT` or equivalent
3. Replace spaces with hyphens
4. Keep only `[a-z0-9-]`

**File slug vs name change:** If the data repo already has a `config.yaml` with a different `file_slug`, warn:
> "Your existing file slug is `<old_slug>` but your name would generate `<new_slug>`. Existing summary files won't be renamed. Continue with new slug?"
> - Yes, use new slug
> - No, keep existing slug

Convert chosen time + days to cron (add a random 1-5 minute offset to avoid exact-hour load):
- Weekdays 5 PM → `3 17 * * 1-5`
- Every day 5 PM → `3 17 * * *`

Write `DATA_REPO_PATH/config.yaml`:

```yaml
# ============================================================
# Daily Summaries — Configuration
# ============================================================
# Edit this file to change how your daily summaries work.
# After saving, the next scheduled run will pick up changes.
# Re-run the setup wizard: "Run the setup skill"
# ============================================================

config_version: 1

# --- Skills repo (public) ---
skills_repo_path: "<SKILLS_REPO_PATH>"   # local path to the cloned public repo

# --- Your identity ---
user:
  full_name: "<full_name>"
  timezone: "<timezone>"
  file_slug: "<file_slug>"
  slack_user_id: "<slack_user_id>"

# --- Google Drive (local sync via Google Drive for Desktop) ---
google_drive:
  local_path: "<drive_path>"   # omit or leave blank if disabled

# --- Schedule ---
schedule:
  cron: "<cron_expression>"
  rollup_day: "friday"

# --- Summary format preferences ---
format:
  sections:
    summary: true
    decisions_and_rationale: <true|false>
    open_loops: <true|false>
    blockers: <true|false>
    next_steps: <true|false>
    transcript_source: <true|false>
  bullet_target: "<bullet_target>"
  yaml_frontmatter: true

# --- Cross-day context ---
context:
  load_previous_day: <true|false>
  lookback_days: <1|2|3>

# --- Data source rules ---
data_sources:
  google_calendar:
    enabled: <true|false>
  gmail:
    enabled: <true|false>
    action_only: <true|false>
  slack:
    enabled: <true|false>
    sent_only: <true|false>
  google_drive:
    enabled: <true|false>
```

Note: `skills_repo_path` is stored in config so the skill always knows where to find companion files if needed.

---

## STEP 9 — Discover MCP connector UUIDs and write permissions

MCP connector tool names contain account-specific UUIDs. Discover them now so scheduled tasks never prompt for permission.

1. Use `ToolSearch` with keyword queries to discover tools:
   - `gcal list events` → Google Calendar tool
   - `gmail search` → Gmail tools
   - `slack search` → Slack tool
   - `google drive search` → Google Drive tool (only if Drive MCP source is enabled)
2. For each enabled source, verify the tool was found. If not:
   > "Could not find the MCP connector for <source>. Make sure it's enabled in Claude Code Settings → MCP Connectors. You can re-run setup after enabling it."
3. Read the existing `SKILLS_REPO_PATH/.claude/settings.local.json` (or start from `{}` if it doesn't exist).
4. Merge the discovered tool permission names into the `permissions.allow` array:
   - `mcp__<uuid>__gcal_list_events`
   - `mcp__<uuid>__gmail_search_messages`, `mcp__<uuid>__gmail_read_message`, `mcp__<uuid>__gmail_read_thread`
   - `mcp__<uuid>__slack_search_public_and_private`
   - `mcp__<uuid>__google_drive_search`, `mcp__<uuid>__google_drive_fetch` (if applicable)
5. Write the merged result back to `SKILLS_REPO_PATH/.claude/settings.local.json`.

---

## STEP 10 — Connectivity test

Run a quick check for each enabled source:

1. **Google Calendar**: Call `gcal_list_events` for today. Any response (even empty) = connected.
2. **Gmail**: Call `gmail_search_messages` with `from:me`. Any response = connected.
3. **Slack**: Call `slack_search_public_and_private` with `from:<@SLACK_USER_ID>`. Any response = connected.
4. **Google Drive (local)**: Run `test -d "<drive_path>"`.

Print results:
```
Connectivity check:
  Google Calendar: ✓ Connected
  Gmail:           ✓ Connected
  Slack:           ✗ Not connected — check MCP connector in Claude Code settings
  Google Drive:    ✓ Local folder found at <path>
```

Continue regardless — the user can fix connectors later.

---

## STEP 11 — Create scheduled tasks

Use `ToolSearch` to load the scheduled task tools:
```
ToolSearch query: "select:mcp__scheduled-tasks__create_scheduled_task,mcp__scheduled-tasks__list_scheduled_tasks,mcp__scheduled-tasks__update_scheduled_task"
```

Detect the default branch: `cd DATA_REPO_PATH && git branch --show-current` → `DATA_BRANCH`
Detect the skills branch: `cd SKILLS_REPO_PATH && git branch --show-current` → `SKILLS_BRANCH`

Check existing tasks with `mcp__scheduled-tasks__list_scheduled_tasks`.

### Daily work summary task

If `daily-work-summary` already exists, ask:
> "A 'daily-work-summary' scheduled task already exists. Replace it with new settings?"
> - Yes, update it
> - No, keep existing

Use `mcp__scheduled-tasks__create_scheduled_task` (or `update_scheduled_task`) with:
- `taskId`: `daily-work-summary`
- `description`: `Gather today's work activity and produce a structured daily summary`
- `cronExpression`: the cron from Step 7
- `prompt`: see template below

**Daily task prompt template:**
```
## Instructions

SKILLS_REPO_PATH: <SKILLS_REPO_PATH>
DATA_REPO_PATH: <DATA_REPO_PATH>

1. Pull latest skills: change directory to SKILLS_REPO_PATH and run `git pull origin <SKILLS_BRANCH>`.
2. Pull latest data: change directory to DATA_REPO_PATH and run `git pull origin <DATA_BRANCH>`.
3. Read the skill file at: SKILLS_REPO_PATH/.claude/skills/daily-work-summary.md
4. Read config at: DATA_REPO_PATH/config.yaml
5. Execute all steps defined in the skill file exactly as written.
   - DATA_REPO_PATH is your working directory for all file reads and writes.
   - All summary files are read from and written to DATA_REPO_PATH.
   - Commit and push from DATA_REPO_PATH when done.

IMPORTANT: The skill file and config are the single source of truth. Do not use cached instructions from prior runs.
```

### Weekly rollup task (if enabled in Step 7)

Same pattern: check if `weekly-rollup` exists, ask to update or create.

- `taskId`: `weekly-rollup`
- `description`: `Synthesize the week's daily summaries into a weekly rollup`
- `cronExpression`: rollup day at 30 minutes after daily time (e.g., if daily is `3 17 * * 1-5`, rollup is `33 17 * * 5`)
- `prompt`: same template but referencing `.claude/skills/weekly-rollup.md` instead

### Verify

After creating/updating, call `mcp__scheduled-tasks__list_scheduled_tasks` and show the next run time to the user.

---

## STEP 12 — Commit and push data repo

1. `cd DATA_REPO_PATH`
2. `git add config.yaml README.md .gitignore`
3. `git commit -m "Configure daily summaries via setup wizard"`
4. `git push origin <DATA_BRANCH>`

If push fails, warn the user and print the error.

**Migration offer:** If any `*-daily-summary-*.md` or `weekly-rollup-*.md` files exist in `SKILLS_REPO_PATH` (from a previous single-repo setup), ask:
> "Found existing summary files in the skills repo. Move them to your data repo?"
> - Yes, move them
> - No, leave them

If yes: copy the files to `DATA_REPO_PATH`, `git add`, commit with message `"Migrate existing summaries to data repo"`, and push.

---

## STEP 13 — Print summary

```
Setup complete!

Repos:
  Skills (public):  <SKILLS_REPO_PATH>  → <skills_github_url>
  Data   (private): <DATA_REPO_PATH>    → <data_github_url>

Configuration:
  Name:       <full_name>
  Timezone:   <timezone>
  Slug:       <file_slug>
  Schedule:   <human-readable, e.g., "Weekdays at 5:00 PM ET">
  Rollup:     <Enabled / Disabled>
  Sources:    <list>
  Drive:      <path or "Disabled">

Scheduled tasks:
  daily-work-summary    — <cron>  (next run: <datetime>)
  weekly-rollup         — <cron or "Not created">

Connectivity:
  Google Calendar: <✓ / ✗>
  Gmail:           <✓ / ✗>
  Slack:           <✓ / ✗>
  Google Drive:    <✓ / ✗>
```

Then print the checklist for anything that failed:
```
Remaining steps:
  [ ] Enable any MCP connectors that failed the connectivity check
      (Claude Code → Settings → MCP Connectors)
  [ ] Run the daily task once manually to approve any remaining tool permissions
      (Scheduled sidebar → daily-work-summary → Run now)

To re-run setup: open the skills repo in Claude Code and say "Run the setup skill"
To edit config:  edit DATA_REPO_PATH/config.yaml
```

---

## Constraints
- Always validate inputs before writing config.
- Never overwrite config without user confirmation.
- If any step fails, tell the user what went wrong and how to fix it — don't silently skip.
- Keep prompts short and friendly. One question at a time where possible.
- Use `AskUserQuestion` for all user prompts — do not use free-text input except for name, path, and Slack ID.
- Never commit to the skills (public) repo during setup — only to the data (private) repo.
