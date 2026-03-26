---
name: daily-work-summary
description: Gather today's work activity and produce a structured daily summary committed to the private data repo
---

## Daily Work Summary — Automated Run

This skill is executed by a scheduled task that provides two path variables in its context:

- **`SKILLS_REPO_PATH`** — the local path to the public skills repo (where this file lives)
- **`DATA_REPO_PATH`** — the local path to the private data repo (where `config.yaml` and summaries live)

**If either variable is missing or `DATA_REPO_PATH/config.yaml` cannot be read, STOP and output:**
> "ERROR: DATA_REPO_PATH is not set or config.yaml is missing. Re-run the setup skill from the skills repo to reconfigure your scheduled tasks."

Read `DATA_REPO_PATH/config.yaml` at the start of every run. All variables (`USER_FULL_NAME`, `USER_TIMEZONE`, `FILE_SLUG`, `SLACK_USER_ID`, etc.) come from that file.

**Today's date:** Determine the current local date in `USER_TIMEZONE` at the time this task runs.

---

## STEP 0 — Load deferred tools

Before gathering any data, use `ToolSearch` to load the MCP connector tools needed for data gathering (only for sources enabled in `config.yaml` → `data_sources`):
- `gcal list events` → Google Calendar tool
- `gmail search` → Gmail tools
- `slack search` → Slack search tool

If a ToolSearch query returns no results for an enabled source, note it in the completion report as "MCP connector not available — check Claude Code Settings → MCP Connectors."

**General resilience:**
- If a tool becomes available mid-run (e.g., via a system reminder), use it immediately — do NOT restart from scratch.
- Retry failed tool calls up to 3 times before skipping a source.
- If the session resumes after an interruption, pick up from where you left off.

---

## STEP 1 — Load cross-day context

Check `config.yaml` → `context.load_previous_day`. If `true`:

1. Determine the lookback window: use `context.lookback_days` (default: 1).
2. Use `Glob` with pattern `FILE_SLUG-daily-summary-*.md` inside `DATA_REPO_PATH` to find all existing summaries. Select the most recent ones within `lookback_days` calendar days.
3. For each found summary, read it and extract:
   - **Open Loops** → carry forward if still unresolved
   - **Next Steps** → check if today's activity resolves them
   - **Tags** from frontmatter → continuity
4. If no summaries are found within the window, expand the search up to 10 calendar days back to find the most recent summary. Note the gap in the completion report.

If no previous summary is found at all, proceed without cross-day context (do not error).

---

## STEP 2 — Gather data

Use only verified, connected sources. Do NOT use internet search or invent details. Only include data from TODAY in `USER_TIMEZONE`. Check `config.yaml` → `data_sources`.

### Google Calendar (if `data_sources.google_calendar.enabled`)
- Use `gcal_list_events` to fetch all events for today.
- Note title, time, attendees, and whether `USER_FULL_NAME` was organizer or attendee.
- Flag events with meeting notes / transcript linked in the description.

### Gmail (if `data_sources.gmail.enabled`)
- Use `gmail_search_messages` to find emails where `USER_FULL_NAME` **sent** a reply or took action today (e.g., `from:me after:<today> before:<tomorrow>`).
- If `data_sources.gmail.action_only` is `true`: **DO NOT summarize emails received-only without action.**

### Slack (if `data_sources.slack.enabled`)
- Use `slack_search_public_and_private` to find messages sent by `USER_FULL_NAME` today. Query: `from:<@SLACK_USER_ID> on:<today>`, sort by `timestamp`.
- If `data_sources.slack.sent_only` is `true`: **DO NOT summarize received-only messages.**
- Note: channel/DM name, message content summary, decisions made, action items.

### Google Drive (if `data_sources.google_drive.enabled`)
Files are accessed via a locally-synced folder (Google Drive for Desktop). Path is in `config.yaml` → `google_drive.local_path`.

1. Use `Glob` to scan the local Drive folder for files modified today: `**/*` within `local_path`.
2. Use `Bash` with `find` if needed: `find "<local_path>" -type f -newermt "<YYYY-MM-DD>" ! -newermt "<YYYY-MM-DD+1>"`
3. For each file: note name, type, subfolder path, and whether created vs edited.
4. If the local path doesn't exist, note "Google Drive folder not synced" in the completion report.

---

## STEP 3 — Compose the raw narrative

Write a natural first-person narrative (2–5 sentences per topic) summarizing today as if `USER_FULL_NAME` had dictated it:
- Meetings attended (from Calendar)
- Actions taken on emails
- Notable Slack messages (decisions, action items, announcements)
- Google Drive files created or meaningfully edited
- Cross-tool patterns (e.g., a Drive file discussed in a meeting)

This becomes the **Transcript Source (Cleaned)** section.

---

## STEP 4 — Apply Daily Summary instructions

Using the narrative from Step 3 and cross-day context from Step 1, produce the structured summary.

**Summarization rules:**
- Group related activity — do not create one bullet per Slack message.
- Format: Who + What + Where/Tool + Why/Result.
- Include people, tools, projects, locations.
- One sentence max per bullet. Omit filler exchanges.
- Only capture: decisions made, things sent/shared, work completed, commitments given.
- Add `(transcript available)` if a Calendar event had a linked transcript.
- Add `(transcript not available)` if a meeting is mentioned but no transcript found.
- No vague commentary. No invented details.
- Target bullet count: `config.yaml` → `format.bullet_target` (default: 5-15).

**Sections** (check `config.yaml` → `format.sections` for which to include):

### Decisions & Rationale
- Decisions made today with brief rationale.
- Format: `- **<Decision>**: <Why / context>`

### Open Loops
- Unresolved items waiting on someone/something.
- From cross-day context: if still unresolved, carry forward with `(carried from <YYYY-MM-DD>)`.
- Items carried for more than 5 working days (count actual summary files in DATA_REPO_PATH since first opened) → tag `(stale — consider closing or escalating)`.

### Blockers
- Anything actively blocking progress today.

### Next Steps
- Planned or implied actions for tomorrow or near-term.

**Auto-tagging:** Derive 3-8 lowercase tags from content (project names, tool names, topic keywords). These go into YAML frontmatter.

---

## STEP 5 — Assemble the .md file content

Check `config.yaml` → `format.yaml_frontmatter`. If `true`, start with:

```
---
date: "<YYYY-MM-DD>"
weekday: "<Full weekday name>"
author: "<USER_FULL_NAME>"
quality: "<complete|partial|manual-only>"
sources_used:
  - <list of sources that returned data>
sources_empty:
  - <list of sources that returned no data>
open_loops_carried: <count from yesterday, 0 if none>
tags:
  - <auto-derived tags>
---
```

Then the markdown body with sections enabled in `config.yaml` → `format.sections`.

---

## STEP 6 — Check for existing file and merge

Check if `DATA_REPO_PATH/FILE_SLUG-daily-summary-<YYYY-MM-DD>.md` already exists.

If found:
- Compare `sources_used` in existing frontmatter with current run's sources. If all sources already listed and quality is `complete`, note in the report ("Merging into existing complete file").
- Merge existing + new bullets in each section. Remove semantic duplicates. Keep the complete set.
- Aggregate Transcript Source sections (deduplicate substantially overlapping content).
- Merge frontmatter: combine `sources_used`, update `quality`, merge `tags`.

If not found: use content from Step 5 as-is.

---

## STEP 7 — Write file and commit to DATA_REPO_PATH

1. Write the final content to: `DATA_REPO_PATH/FILE_SLUG-daily-summary-<YYYY-MM-DD>.md`
2. Detect the default branch: `cd DATA_REPO_PATH && git branch --show-current`
3. Pull latest: `git pull origin <branch>`
4. Stage: `git add FILE_SLUG-daily-summary-<YYYY-MM-DD>.md`
5. Commit: `git commit -m "daily summary <YYYY-MM-DD>"`
6. Push: `git push origin <branch>`

---

## STEP 7b — Copy to Google Drive (if configured)

Check `config.yaml` → `outputs.google_drive_copy.enabled`. If `true`:

1. Read `outputs.google_drive_copy.path` as `GDRIVE_COPY_PATH`.
2. Verify the path exists: `test -d "<GDRIVE_COPY_PATH>"`.
   - If it does not exist, attempt to create it: `mkdir -p "<GDRIVE_COPY_PATH>"`.
   - If creation fails, note "Google Drive copy folder could not be created at `<GDRIVE_COPY_PATH>`" in the completion report and skip this step.
3. Copy the summary file:
   - `cp "DATA_REPO_PATH/FILE_SLUG-daily-summary-<YYYY-MM-DD>.md" "GDRIVE_COPY_PATH/FILE_SLUG-daily-summary-<YYYY-MM-DD>.md"`
4. Confirm in the completion report: "Copied to Google Drive: `GDRIVE_COPY_PATH/FILE_SLUG-daily-summary-<YYYY-MM-DD>.md`"

If `outputs.google_drive_copy.enabled` is `false` or the key is absent, skip silently.

---

## STEP 8 — Display summary and completion report

**Always display the full summary** in task output so the user can read it without opening the file.

Output the complete file content (frontmatter + all sections), then a short completion report:
- Date processed
- Bullets: summary count, decisions, open loops, blockers, next steps
- Whether an existing file was found and merged
- Cross-day context loaded (from which date)
- Confirmation that the file was committed and pushed to `DATA_REPO_PATH`
- Whether a Google Drive copy was made (path) or skipped/failed
- Data sources that returned no results or were unavailable
- Whether `config.yaml` was read successfully

---

## Constraints
- Only use data from TODAY in `USER_TIMEZONE`.
- Do NOT summarize received-only emails or Slack messages (when action_only/sent_only is enabled).
- Do NOT invent details or use internet search.
- All file writes and commits go to `DATA_REPO_PATH`, never to `SKILLS_REPO_PATH`.
- Preserve named entities: people, tools, project names, locations.
- Filename date must match the header date exactly.
