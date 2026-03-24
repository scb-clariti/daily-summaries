---
name: daily-work-summary
description: Gather today's work activity from Clariti stack and produce a structured daily summary .md file committed to this repo
---

## Daily Work Summary — Automated Run

**User:** Samuel Couture Brochu
**Timezone:** Eastern (America/Toronto)
**Today's date:** Determine the current local date in America/Toronto at the time this task runs.

---

## STEP 1 — Gather data from the Clariti stack

Use only verified, connected sources. Do NOT use internet search or invent details. Only include data from TODAY (the current local date in America/Toronto).

### Google Calendar
- Use `gcal_list_events` to fetch all events for today.
- For each event: note title, time, attendees, and whether Samuel was an organizer or attendee.
- Flag events that have a meeting notes / transcript attached or linked in the description.

### Gmail
- Use `gmail_search_messages` to find emails where Samuel **sent** a reply or performed an action today (e.g., search `from:me after:<today> before:<tomorrow>`).
- Also search for emails where Samuel explicitly took an action (forwarded, flagged, responded).
- **DO NOT summarize emails Samuel only received and did not act on.**

### Jira
- Use `searchJiraIssuesUsingJql` to find issues updated or transitioned by Samuel today. Suggested JQL: `assignee = currentUser() AND updated >= startOfDay() ORDER BY updated DESC` and also `reporter = currentUser() AND created >= startOfDay()`.
- Note: issue key, summary, status transition (if any), and any comments Samuel added today.

### Confluence
- Use `searchConfluenceUsingCql` to find pages created or edited by Samuel today. Suggested CQL: `contributor = currentUser() AND lastModified >= startOfDay()`.
- Note: page title, space, and nature of the change.

---

## STEP 2 — Compose the raw narrative (dictation stand-in)

Write a natural first-person-style narrative paragraph (2–5 sentences per topic) summarizing what happened today, as if Samuel had dictated it. Cover:
- Meetings attended (from Calendar)
- Actions taken on emails
- Jira work (tickets updated, transitioned, commented on)
- Confluence pages created/updated
- Any notable cross-tool patterns (e.g., a Jira ticket discussed in a meeting)

This narrative becomes the **Transcript Source (Cleaned)** for the summary instructions below.

---

## STEP 3 — Apply Daily Summary instructions

Using the narrative from Step 2 as input, produce the structured daily summary.

**Variables:**
- USER_FULL_NAME: Samuel Couture Brochu
- USER_TIMEZONE: Eastern (America/Toronto)
- Date slug: `samuel-couture-brochu`

**Summarization rules:**
- One bullet per event/action/topic.
- Format: Who + What + Where/Tool + Why/Result.
- Include people, tools, projects, locations.
- Add `(transcript available)` if a Calendar event had a linked transcript/notes.
- Add `(transcript not available)` if a meeting is mentioned but no transcript found.
- No vague commentary. No invented details.

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

Check if `samuel-couture-brochu-daily-summary-<YYYY-MM-DD>.md` already exists in the repository root.

If an existing file is found:
- Read its current bullets (summary block).
- Merge existing bullets + new bullets. Remove duplicates. Keep the complete merged set.
- Aggregate existing Transcript Source + new narrative into a single Transcript Source section.

If no existing file is found, use the content from Step 4 as-is.

---

## STEP 6 — Write the file and commit

1. Write the final merged content to: `samuel-couture-brochu-daily-summary-<YYYY-MM-DD>.md` in the repository root.
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
- Only use data from TODAY in America/Toronto timezone.
- Do NOT summarize emails Samuel only received without acting on them.
- Do NOT invent details or use internet search.
- Save files to the repo and commit/push.
- Preserve named entities: people, tools, project names, locations.
- Remove filler words ("um", "like") from any dictated/natural-language sources if applicable.
- If a date or weekday is ambiguous, compute it from the current timestamp — do not guess.
- Filename date must match the header date exactly.
