---
name: daily-work-summary
description: Gather today's work activity from Clariti stack and produce a structured daily summary .md file committed to this repo
---

## Variables

- USER_FULL_NAME: `Samuel Couture Brochu`
- USER_TIMEZONE: `America/Toronto`
- FILE_SLUG: `samuel-couture-brochu`
- GDRIVE_ARTIFACTS_FOLDER_PATH: `My Drive/personal-ai-artifacts`
- GDRIVE_ARTIFACTS_FOLDER_ID: `1_wGiMS2YN_2G8udVC_EKXI00Tw385qC5`
- SLACK_USER_ID: `U0AFFB01MNE`

---

## Daily Work Summary — Automated Run

**User:** `USER_FULL_NAME`
**Timezone:** `USER_TIMEZONE`
**Today's date:** Determine the current local date in `USER_TIMEZONE` at the time this task runs.

---

## STEP 0 — Load deferred tools

Before gathering any data, use `ToolSearch` to load the following tools so they are available for subsequent steps:
- `google_drive_search` and `google_drive_fetch` — use query: `select:mcp__c1fc4002-5f49-5f9d-a4e5-93c4ef5d6a75__google_drive_search,mcp__c1fc4002-5f49-5f9d-a4e5-93c4ef5d6a75__google_drive_fetch`
- `slack_search_public_and_private` — use query: `select:mcp__6ec975e1-fbf4-470a-beec-a0ec45c64a40__slack_search_public_and_private`

This is required because these are deferred tools that are not active until explicitly fetched. Always use the `select:` syntax with full tool names to avoid keyword matching failures.

---

## STEP 1 — Gather data from the Clariti stack

Use only verified, connected sources. Do NOT use internet search or invent details. Only include data from TODAY (the current local date in `USER_TIMEZONE`).

### Google Calendar
- Use `gcal_list_events` to fetch all events for today.
- For each event: note title, time, attendees, and whether `USER_FULL_NAME` was an organizer or attendee.
- Flag events that have a meeting notes / transcript attached or linked in the description.

### Gmail
- Use `gmail_search_messages` to find emails where `USER_FULL_NAME` **sent** a reply or performed an action today (e.g., search `from:me after:<today> before:<tomorrow>`).
- Also search for emails where `USER_FULL_NAME` explicitly took an action (forwarded, flagged, responded).
- **DO NOT summarize emails `USER_FULL_NAME` only received and did not act on.**

### Slack
- Use `slack_search_public_and_private` to find messages sent by `USER_FULL_NAME` today. Suggested query: `from:<@SLACK_USER_ID> on:<today>`, sort by `timestamp`.
- Include messages sent in channels, DMs, and threads.
- **DO NOT summarize messages `USER_FULL_NAME` only received and did not respond to.**
- Note: channel/DM name, message content summary, and any notable context (e.g., decision made, action item assigned).

### Google Drive
- Use `google_drive_search` to find files created or modified by `USER_FULL_NAME` today within `GDRIVE_ARTIFACTS_FOLDER_ID` (`GDRIVE_ARTIFACTS_FOLDER_PATH`) and all nested subfolders. Use a breadth-first recursive approach:
  1. Query direct children of `GDRIVE_ARTIFACTS_FOLDER_ID`: `modifiedTime > '<today>T00:00:00' and '<GDRIVE_ARTIFACTS_FOLDER_ID>' in parents`
  2. Collect any subfolders (mimeType = `application/vnd.google-apps.folder`) returned.
  3. For each subfolder, repeat the query with that subfolder's ID as the parent.
  4. Continue until no new subfolders are found.
- Note: file name, type, full subfolder path, and nature of the change (created vs. edited).

---

## STEP 2 — Compose the raw narrative (dictation stand-in)

Write a natural first-person-style narrative paragraph (2–5 sentences per topic) summarizing what happened today, as if `USER_FULL_NAME` had dictated it. Cover:
- Meetings attended (from Calendar)
- Actions taken on emails
- Slack messages sent (notable decisions, action items, announcements)
- Google Drive files created or meaningfully edited
- Any notable cross-tool patterns (e.g., a Drive file discussed in a meeting or Slack thread)

This narrative becomes the **Transcript Source (Cleaned)** for the summary instructions below.

---

## STEP 3 — Apply Daily Summary instructions

Using the narrative from Step 2 as input, produce the structured daily summary.

**Summarization rules:**
- Group related activity into single bullets — do not create one bullet per Slack message.
- Format: Who + What + Where/Tool + Why/Result.
- Include people, tools, projects, locations.
- Keep bullets concise: one sentence max. Omit filler exchanges (acknowledgements, one-word replies).
- Only capture substantive actions: decisions made, things sent/shared, work completed, commitments given.
- Add `(transcript available)` if a Calendar event had a linked transcript/notes.
- Add `(transcript not available)` if a meeting is mentioned but no transcript found.
- No vague commentary. No invented details.
- Target: 5–10 bullets total for a typical day.

**Output format** for the summary block:

<Weekday> - <YYYY-MM-DD>:
- <item 1>
- <item 2>
- <item n>

---

## STEP 4 — Assemble the full .md file content

Combine the summary block and the cleaned transcript:

<Weekday> - <YYYY-MM-DD>:
- <item 1>
- <item n>

## Transcript Source (Cleaned)
<narrative from Step 2>

---

## STEP 5 — Check for existing file in the repo and merge

Check if `FILE_SLUG-daily-summary-<YYYY-MM-DD>.md` already exists in the repository root.

If an existing file is found:
- Read its current bullets (summary block).
- Merge existing bullets + new bullets. Remove duplicates. Keep the complete merged set.
- Aggregate existing Transcript Source + new narrative into a single Transcript Source section.

If no existing file is found, use the content from Step 4 as-is.

---

## STEP 6 — Write the file and commit

1. Write the final merged content to: `FILE_SLUG-daily-summary-<YYYY-MM-DD>.md` in the repository root.
2. Stage the file with `git add`.
3. Commit with message: `daily summary <YYYY-MM-DD>`
4. Push to origin.

---

## STEP 7 — Output a completion report

Print a short summary:
- Date processed
- Number of bullets in the summary
- Whether an existing file was found and merged
- Confirmation that the file was committed and pushed
- List any data sources that returned no results (so it's visible if a tool failed)

---

## Constraints & preferences
- Only use data from TODAY in `USER_TIMEZONE` timezone.
- Do NOT summarize emails `USER_FULL_NAME` only received without acting on them.
- Do NOT invent details or use internet search.
- Save files to the repo and commit/push.
- Preserve named entities: people, tools, project names, locations.
- Remove filler words ("um", "like") from any dictated/natural-language sources if applicable.
- If a date or weekday is ambiguous, compute it from the current timestamp — do not guess.
- Filename date must match the header date exactly.
