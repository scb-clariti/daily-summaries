---
name: setup
description: Interactive setup wizard for Daily Summaries — configures identity, creates the private data repo, sets up scheduled tasks, and pushes everything to GitHub
---

## Daily Summaries — Setup Wizard

This skill walks you through a one-time setup. By the end you will have:
- A place for your summaries — either a **private GitHub repo** or a **Google Drive folder**
- **Scheduled tasks** that pull skills from this repo and write summaries to your data location
- All MCP connectors tested and permissions granted

---

## STEP 0 — Detect environment

### OS & defaults
1. Run `uname -s` to detect OS:
   - `Darwin` → macOS
   - `MINGW*`, `MSYS*`, `CYGWIN*` → Windows (Git Bash)
2. Run `git config user.name` to get the default name (if git is available; skip if not).
3. Detect timezone:
   - macOS: `readlink /etc/localtime | sed 's|.*/zoneinfo/||'`
   - Windows: `powershell -Command "[System.TimeZoneInfo]::Local.Id"` then map to IANA (e.g., "Eastern Standard Time" → "America/Toronto", "Pacific Standard Time" → "America/Los_Angeles", "Central Standard Time" → "America/Chicago", "Mountain Standard Time" → "America/Denver")

### Detect SKILLS_REPO_PATH
4. Run `pwd` in Bash. This is `SKILLS_REPO_PATH` — the directory where this setup wizard is running (the cloned public repo).
5. If git is available, run `git rev-parse --is-inside-work-tree` to confirm we are inside the skills repo.

### Pre-flight: GitHub + Google Drive availability
6. **GitHub check**: Run `gh --version` to check if `gh` is installed, then `gh auth status` to check authentication. Record result as `GH_AVAILABLE = true | false`.
7. **Google Drive check**: Auto-detect Google Drive for Desktop at common paths:
   - macOS: `~/Library/CloudStorage/GoogleDrive-*/My Drive` or `~/Google Drive/My Drive`
   - Windows: `C:\Users\<username>\My Drive`, `G:\My Drive`, `C:\Users\<username>\Google Drive`
   - Validate with `test -d`. Record result as `GDRIVE_AVAILABLE = true | false` and store the detected path.
8. **Gate check**: If `GH_AVAILABLE = false` AND `GDRIVE_AVAILABLE = false`:
   > "Setup requires at least one storage destination for your summaries — either a GitHub account or Google Drive for Desktop. Please set up one of the following, then re-run setup:"
   > ```
   > Option A — GitHub:
   >   macOS:   brew install gh
   >   Windows: winget install --id GitHub.cli
   >   Then:    gh auth login
   >
   > Option B — Google Drive for Desktop:
   >   Install from https://www.google.com/drive/download/
   > ```
   Stop.
9. **Set STORAGE_MODE**:
   - If `GH_AVAILABLE = true`: set `STORAGE_MODE = github` (GitHub is the primary storage; Google Drive copy is optional later).
   - If `GH_AVAILABLE = false` and `GDRIVE_AVAILABLE = true`: set `STORAGE_MODE = drive` (summaries are written directly to the Google Drive folder — no git).

---

## STEP 1 — Existing config check

Check if a sibling data repo already exists by looking for directories named `*-journal` or `*-daily*` next to `SKILLS_REPO_PATH`. Also check if `config.yaml` exists anywhere nearby. If `STORAGE_MODE = drive`, also look for `config.yaml` inside the detected Google Drive path (e.g., `<drive_path>/daily-summaries/config.yaml`).

If a previous setup is detected, ask:
> "It looks like you may have run setup before. What would you like to do?"
> - Start fresh — create a new data repo and overwrite config
> - Re-run setup on existing data repo — update config only

If "Re-run on existing": ask for the `DATA_REPO_PATH` (the local path to the existing data repo or Drive folder), load `config.yaml` from there as defaults, and skip STEP 3 (data repo creation).

### Migration: Drive → GitHub

If a previous config is found with `storage.mode: drive` AND `GH_AVAILABLE = true` (i.e., the user now has GitHub set up), ask:
> "Your previous setup saved summaries to Google Drive only. You now have GitHub available. Would you like to migrate to a private GitHub repo?"
> - **Yes, migrate to GitHub** — create a private repo and copy existing summaries into it
> - **No, keep using Google Drive**

If **Yes**: set `STORAGE_MODE = github`. After the new GitHub data repo is created in Step 3, copy all existing `*-daily-summary-*.md` and `weekly-rollup-*.md` files from the Drive folder to `DATA_REPO_PATH`, then `git add`, commit with message `"Migrate existing summaries from Google Drive"`, and push.

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

## STEP 3 — Create the data storage

### GitHub path (STORAGE_MODE = github)

Ask:
> "What should your private data repo be named?"
> Options:
> - `daily-summaries-journal` (Recommended)
> - Other (specify)

Ask:
> "Which GitHub account should it be created under?"
> Offer the account detected from `gh auth status` as default.

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

Set `DATA_REPO_PATH` = the absolute local path to the cloned repo.

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

### Google Drive path (STORAGE_MODE = drive)

The Google Drive path was already detected in Step 0. Use it to create the summaries folder.

Ask:
> "Which folder in Google Drive should summaries be saved to?"
> Default: `<detected_drive_path>/daily-summaries`
> (Enter a full absolute path, or press Enter for the default)

1. Run `mkdir -p "<chosen_path>"` to create the folder if it doesn't exist.
2. Validate with `test -d "<chosen_path>"`.

Set `DATA_REPO_PATH` = the chosen Google Drive folder path.

Create a `README.md` in `DATA_REPO_PATH`:
```markdown
# Daily Summaries — Private Journal

This folder contains generated daily work summaries and configuration.
It is managed automatically by Claude Code scheduled tasks and synced via Google Drive.

Do not edit files here manually unless you know what you're doing.
```

---

## STEP 4 — Google Drive for Desktop

**If `STORAGE_MODE = drive`:** Google Drive was already detected and configured in Step 0 and Step 3. Set `google_drive.local_path` to the detected Drive root path, set `outputs.google_drive_copy.enabled: false` (summaries already write directly to Drive — no copy needed). Skip the rest of this step.

**If `STORAGE_MODE = github`:**

Ask:
> "Do you have Google Drive for Desktop installed and syncing?"
> - Yes
> - No — skip Google Drive integration

If yes:
1. Use the Drive path already detected in Step 0 (if `GDRIVE_AVAILABLE = true`). If not detected, auto-detect common paths:
   - macOS: `~/Library/CloudStorage/GoogleDrive-*/My Drive` or `~/Google Drive/My Drive`
   - Windows: `C:\Users\<username>\My Drive`, `G:\My Drive`, `C:\Users\<username>\Google Drive`
2. If found, propose it:
   > "Found Google Drive at: `<path>`. Is this correct?"
   > - Yes
   > - No, let me specify
3. Validate with `test -d`. If it doesn't exist, warn but continue.

If No: set `google_drive.enabled: false`.

### STEP 4b — Google Drive copy destination

Ask:
> "Would you like each summary file copied to a Google Drive folder after it's written? This keeps a synced copy in your Drive alongside your git history."
> - Yes
> - No — skip

If yes:
1. Suggest a sensible default inside the detected Drive path (or prompt freely if Drive was skipped):
   - Default suggestion: `<drive_path>/daily-summaries` (or `~/My Drive/daily-summaries` on macOS)
   > "Which folder should summaries be copied to?"
   > Default: `<suggested_path>`
   > (Enter a full absolute path)
2. Validate with `test -d "<path>"`. If it does not exist:
   > "That folder doesn't exist yet. Create it now?"
   > - Yes → `mkdir -p "<path>"`
   > - No — I'll create it manually later (continue, set `enabled: true` so the skill creates it at runtime)
3. Store as `GDRIVE_COPY_PATH`.
4. Set `outputs.google_drive_copy.enabled: true` and `outputs.google_drive_copy.path: "<GDRIVE_COPY_PATH>"` in the config.

If no: set `outputs.google_drive_copy.enabled: false` and `outputs.google_drive_copy.path: ""`.

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

# --- Storage ---
storage:
  mode: "<github|drive>"       # github = private GitHub repo; drive = Google Drive folder only
  remote_url: "<github_repo_url>"   # blank if drive mode

# --- Your identity ---
user:
  full_name: "<full_name>"
  timezone: "<timezone>"
  file_slug: "<file_slug>"
  slack_user_id: "<slack_user_id>"

# --- Google Drive (local sync via Google Drive for Desktop) ---
google_drive:
  local_path: "<drive_path>"   # omit or leave blank if disabled

# --- Output copies ---
outputs:
  google_drive_copy:
    enabled: <true|false>
    path: "<gdrive_copy_path>"   # absolute path to target folder; leave blank if disabled

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

If `STORAGE_MODE = github`:
- Detect the default branch: `cd DATA_REPO_PATH && git branch --show-current` → `DATA_BRANCH`
- Detect the skills branch: `cd SKILLS_REPO_PATH && git branch --show-current` → `SKILLS_BRANCH`

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
- `prompt`: use the template matching the current `STORAGE_MODE` (see below)

**Daily task prompt template (STORAGE_MODE = github):**
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

**Daily task prompt template (STORAGE_MODE = drive):**
```
## Instructions

SKILLS_REPO_PATH: <SKILLS_REPO_PATH>
DATA_REPO_PATH: <DATA_REPO_PATH>

1. Read the skill file at: SKILLS_REPO_PATH/.claude/skills/daily-work-summary.md
2. Read config at: DATA_REPO_PATH/config.yaml
3. Execute all steps defined in the skill file exactly as written.
   - DATA_REPO_PATH is your working directory for all file reads and writes.
   - All summary files are written directly to DATA_REPO_PATH (a Google Drive folder — no git).
   - Google Drive for Desktop syncs the files automatically.

IMPORTANT: The skill file and config are the single source of truth. Do not use cached instructions from prior runs.
```

### Weekly rollup task (if enabled in Step 7)

Same pattern: check if `weekly-rollup` exists, ask to update or create.

- `taskId`: `weekly-rollup`
- `description`: `Synthesize the week's daily summaries into a weekly rollup`
- `cronExpression`: rollup day at 30 minutes after daily time (e.g., if daily is `3 17 * * 1-5`, rollup is `33 17 * * 5`)
- `prompt`: same template (matching the current `STORAGE_MODE` variant) but referencing `.claude/skills/weekly-rollup.md` instead

### Verify

After creating/updating, call `mcp__scheduled-tasks__list_scheduled_tasks` and show the next run time to the user.

---

## STEP 12 — Save config and finalize

### If STORAGE_MODE = github:

1. `cd DATA_REPO_PATH`
2. `git add config.yaml README.md .gitignore`
3. `git commit -m "Configure daily summaries via setup wizard"`
4. `git push origin <DATA_BRANCH>`

If push fails, warn the user and print the error.

**Migration offer — skills repo:** If any `*-daily-summary-*.md` or `weekly-rollup-*.md` files exist in `SKILLS_REPO_PATH` (from a previous single-repo setup), ask:
> "Found existing summary files in the skills repo. Move them to your data repo?"
> - Yes, move them
> - No, leave them

If yes: copy the files to `DATA_REPO_PATH`, `git add`, commit with message `"Migrate existing summaries to data repo"`, and push.

**Migration offer — Google Drive copy:** If `outputs.google_drive_copy.enabled` is `true`, check how many `*-daily-summary-*.md` and `weekly-rollup-*.md` files exist in `DATA_REPO_PATH`. If any are found, ask:
> "Found <N> existing summary file(s) in your data repo. Copy them to your Google Drive folder now?"
> - Yes, copy all existing summaries
> - No, only copy new ones going forward

If yes:
1. Use `Glob` with pattern `*.md` in `DATA_REPO_PATH` to find all `*-daily-summary-*.md` and `weekly-rollup-*.md` files.
2. Verify `GDRIVE_COPY_PATH` exists (`test -d`); create with `mkdir -p` if needed.
3. For each file, run: `cp "DATA_REPO_PATH/<filename>" "GDRIVE_COPY_PATH/<filename>"`
   - Skip files that already exist at the destination (do not overwrite).
   - Note skipped files in the completion report.
4. Report: "Copied <M> file(s) to `GDRIVE_COPY_PATH`. Skipped <K> already present."

### If STORAGE_MODE = drive:

1. Verify `config.yaml` and `README.md` were written to `DATA_REPO_PATH` (the Google Drive folder).
2. No git operations — files are synced automatically by Google Drive for Desktop.

---

## STEP 13 — Print summary

**If STORAGE_MODE = github:**
```
Setup complete!

Storage:  GitHub (private repo)
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
  GDrive Copy: <path or "Disabled">

Scheduled tasks:
  daily-work-summary    — <cron>  (next run: <datetime>)
  weekly-rollup         — <cron or "Not created">

Connectivity:
  Google Calendar: <✓ / ✗>
  Gmail:           <✓ / ✗>
  Slack:           <✓ / ✗>
  Google Drive:    <✓ / ✗>
```

**If STORAGE_MODE = drive:**
```
Setup complete!

Storage:  Google Drive (no GitHub)
  Skills:     <SKILLS_REPO_PATH>
  Summaries:  <DATA_REPO_PATH>  (synced via Google Drive for Desktop)

Configuration:
  Name:       <full_name>
  Timezone:   <timezone>
  Slug:       <file_slug>
  Schedule:   <human-readable, e.g., "Weekdays at 5:00 PM ET">
  Rollup:     <Enabled / Disabled>
  Sources:    <list>

Scheduled tasks:
  daily-work-summary    — <cron>  (next run: <datetime>)
  weekly-rollup         — <cron or "Not created">

Connectivity:
  Google Calendar: <✓ / ✗>
  Gmail:           <✓ / ✗>
  Slack:           <✓ / ✗>
  Google Drive:    <✓ / ✗>

Note: Summaries are saved to Google Drive only. To migrate to a
private GitHub repo later, install gh (brew install gh), authenticate
(gh auth login), and re-run setup.
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
