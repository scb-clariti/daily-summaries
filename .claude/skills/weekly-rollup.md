---
name: weekly-rollup
description: Synthesize the week's daily summaries into a weekly rollup committed to the private data repo
---

## Weekly Rollup — Automated Run

This skill is executed by a scheduled task that provides two path variables in its context:

- **`SKILLS_REPO_PATH`** — the local path to the public skills repo
- **`DATA_REPO_PATH`** — the local path to the private data repo (where `config.yaml` and summaries live)

**If either variable is missing or `DATA_REPO_PATH/config.yaml` cannot be read, STOP and output:**
> "ERROR: DATA_REPO_PATH is not set or config.yaml is missing. Re-run the setup skill from the skills repo to reconfigure your scheduled tasks."

Read `DATA_REPO_PATH/config.yaml` at the start of every run. All variables (`USER_FULL_NAME`, `USER_TIMEZONE`, `FILE_SLUG`) come from that file.

**This week:** Determine the current ISO week number and year from the current local date in `USER_TIMEZONE`.

---

## STEP 1 — Locate this week's daily summaries

1. Determine the date range for the current ISO week (Monday–Sunday).
2. Check `config.yaml` → `schedule.cron` to determine which days summaries are expected. Parse the day-of-week field:
   - `1-5` = weekdays only → expect 5 summaries
   - `*` or `0-6` = every day → expect 7 summaries
3. Use `Glob` with pattern `FILE_SLUG-daily-summary-*.md` inside `DATA_REPO_PATH` to find all matching files. Filter to the current week's date range.
4. Note which expected days are missing.

If zero daily summaries are found, output a warning and stop.

**Race condition note:** If this rollup runs on the same day as the daily summary task, today's file may not yet exist. Check once — if missing, wait 2 minutes and check again. If still missing after the retry, proceed without it and note "Today's daily summary was not yet available at rollup time."

---

## STEP 2 — Synthesize the weekly rollup

Read all found daily summaries from `DATA_REPO_PATH` and produce:

### Themes & Focus Areas
- 3-7 high-level themes grouping the week's activity (e.g., "Hiring", "SOC2 Compliance", "Q2 OKR Planning").
- For each theme: 1-2 sentence summary of what happened across the week.

### Key Accomplishments
- 5-10 bullets covering the most significant things completed or decided this week.
- Format: What + Impact/Outcome.

### Decisions Made
- Aggregate all Decisions & Rationale sections from the dailies.
- Deduplicate and group by theme.

### Open Loops (End of Week)
- Aggregate all unresolved Open Loops from the latest available daily.
- Flag anything carried for more than 5 working days as stale.

### Carried Into Next Week
- Items from Next Steps and Open Loops that were not resolved.
- These become context for next week's daily summaries.

### Meetings & Collaboration
- Count of meetings attended.
- Key people collaborated with.

### Data Source Coverage
- For each expected day, note which sources were used and which were empty/unavailable.
- Flag any systemic gaps (e.g., "Gmail returned no data all week — check connector").

---

## STEP 3 — Assemble the rollup file

**Filename:** `DATA_REPO_PATH/weekly-rollup-<YYYY>-W<WW>.md`

**Content:**

```markdown
---
week: "<YYYY>-W<WW>"
date_range: "<Monday YYYY-MM-DD> to <Sunday YYYY-MM-DD>"
author: "<USER_FULL_NAME>"
days_covered: <count>
days_expected: <expected_days>
days_missing:
  - <YYYY-MM-DD>
tags:
  - <aggregated unique tags from all dailies>
---

# Weekly Rollup — <YYYY>-W<WW>

**<Monday YYYY-MM-DD> to <Sunday YYYY-MM-DD>** | **<count>/<expected_days> days covered**

## Themes & Focus Areas
- **<Theme>**: <summary>

## Key Accomplishments
- <accomplishment>

## Decisions Made
- **<Decision>**: <rationale>

## Open Loops (End of Week)
- <unresolved item>

## Carried Into Next Week
- <item>

## Meetings & Collaboration
- <count> meetings attended
- Key collaborators: <names>

## Data Source Coverage
| Day | Calendar | Gmail | Slack | Drive |
|-----|----------|-------|-------|-------|
| Mon | ...      | ...   | ...   | ...   |
```

---

## STEP 4 — Write file and commit to DATA_REPO_PATH

1. Write the rollup to `DATA_REPO_PATH/weekly-rollup-<YYYY>-W<WW>.md`.
2. Detect the default branch: `cd DATA_REPO_PATH && git branch --show-current`.
3. Pull latest: `git pull origin <branch>`.
4. Stage: `git add weekly-rollup-<YYYY>-W<WW>.md`.
5. Commit: `git commit -m "weekly rollup <YYYY>-W<WW>"`
6. Push: `git push origin <branch>`.

---

## STEP 4b — Copy to Google Drive (if configured)

Check `config.yaml` → `outputs.google_drive_copy.enabled`. If `true`:

1. Read `outputs.google_drive_copy.path` as `GDRIVE_COPY_PATH`.
2. Verify the path exists: `test -d "<GDRIVE_COPY_PATH>"`.
   - If it does not exist, attempt to create it: `mkdir -p "<GDRIVE_COPY_PATH>"`.
   - If creation fails, note "Google Drive copy folder could not be created at `<GDRIVE_COPY_PATH>`" in the completion report and skip this step.
3. Copy the rollup file:
   - `cp "DATA_REPO_PATH/weekly-rollup-<YYYY>-W<WW>.md" "GDRIVE_COPY_PATH/weekly-rollup-<YYYY>-W<WW>.md"`
4. Confirm in the completion report: "Copied to Google Drive: `GDRIVE_COPY_PATH/weekly-rollup-<YYYY>-W<WW>.md`"

If `outputs.google_drive_copy.enabled` is `false` or the key is absent, skip silently.

---

## STEP 5 — Completion report

Print:
- Week processed (ISO week + date range)
- Daily summaries found / expected (which days were missing)
- Themes, accomplishments, decisions, open loops count
- Confirmation committed and pushed to `DATA_REPO_PATH`
- Warnings: missing days, stale open loops, data source gaps

---

## Constraints
- Only use data from this week's daily summary files in `DATA_REPO_PATH` — do NOT query MCP connectors directly.
- Do NOT invent details. If a daily summary is missing, note it and work with what's available.
- Keep the rollup concise — it's a synthesis, not a concatenation.
- All file writes and commits go to `DATA_REPO_PATH`, never to `SKILLS_REPO_PATH`.
- Filename week number must match the content's date range exactly.
