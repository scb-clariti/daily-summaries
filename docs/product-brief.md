# Product Brief: Daily Summaries

**Date:** 2026-03-25
**Version:** 1.0
**Project Type:** Automation / Personal Knowledge Management
**Project Level:** 1

---

## Executive Summary

An automated system that gathers a user's daily work activity from connected tools (Google Calendar, Gmail, Slack, Google Drive), synthesizes it into a structured daily summary, and commits the result to a version-controlled repository. It uses Claude Code scheduled tasks with MCP connectors, eliminating manual dictation while producing LLM-optimized context files for both the user's future self and AI agents.

---

## Problem Statement

### The Problem

Maintaining a daily record of work activity requires manual effort — writing notes at end of day, managing files across tools, and manually maintaining context across sessions. Existing workflows have multiple moving parts that are fragile and require the user to remember to provide input every day.

### Why Now?

Claude Code now offers scheduled tasks with direct access to common work tools (Google Calendar, Gmail, Slack) via MCP connectors, making full automation possible.

### Impact if Unsolved

- Daily context is lost when the user forgets to log or the workflow breaks.
- AI agents in future sessions lack structured context about prior work, reducing their effectiveness.
- Multi-day projects lose continuity; decisions and rationale evaporate.
- The user spends 10-15 minutes daily on a task that should be zero-effort.

---

## Target Audience

### Primary Users

- **Knowledge workers** — Anyone who uses Google Calendar, Gmail, Slack, and Google Drive daily and wants a passive, zero-effort record of daily work.

### Secondary Users

- **AI agents** (Claude Code sessions, future LLM-based tools) — Consume the daily summary files as context to understand what the user has been working on, who they're interacting with, what decisions were made, and what's pending.

### User Needs

1. **Zero-effort daily capture** — The system should run automatically without any manual input on typical days.
2. **Accurate, structured context** — Summaries must be factual, specific (names, ticket IDs, decisions), and formatted for both human scanning and LLM consumption.
3. **Continuity across days** — The system should track open loops, carry forward unresolved items, and support multi-day narrative arcs.

---

## Solution Overview

### Proposed Solution

A Claude Code scheduled task that runs on a user-configured schedule. It queries all connected work tools for the day's activity, composes a natural-language narrative, structures it into a standardized daily summary, and commits the resulting `.md` file to a private GitHub repository.

### Key Features

- Automated data gathering from Google Calendar, Gmail, Slack, and Google Drive
- Natural-language narrative composition (as if the user had dictated it)
- Structured summary with consistent bullet format (Who + What + Where/Tool + Why/Result)
- Merge logic for idempotent runs (existing file + new data = deduplicated merged result)
- Git-based storage with full version history
- Transcript source preservation for raw context
- Interactive setup wizard for zero-config onboarding

### Value Proposition

Eliminates all manual effort from daily work logging while producing higher-quality, more complete summaries than manual dictation — because it pulls from every connected tool rather than relying on memory.

---

## Business Objectives

### Goals

- Achieve fully automated daily summary generation with zero manual input on standard workdays
- Produce summaries that are useful as LLM context within 1 week of initial deployment
- Build a persistent, version-controlled knowledge base of daily work activity

### Success Metrics

- Task runs successfully on all scheduled days without intervention
- Each summary captures all substantive work activity (meetings, emails, messages, decisions)
- Summary files are usable as context in new Claude Code sessions
- Time spent on daily logging drops from ~10-15 min/day to 0 min/day

### Business Value

- ~50-75 minutes/week saved on manual logging
- Richer context for AI agents = better assistance in future sessions
- Searchable work history for performance reviews, handoffs, and retrospectives

---

## Scope

### In Scope

- Automated daily data gathering from: Google Calendar, Gmail, Slack, Google Drive
- Structured daily summary generation (bullets + transcript source)
- Git-based file storage in a private repo (created during setup)
- Merge logic for re-runs and manual additions
- Scheduled execution on user-configured days/time
- YAML frontmatter with date, tags, and metadata
- Sections for: Summary bullets, Decisions & Rationale, Open Loops, Blockers, Next Steps, Transcript Source
- Interactive setup wizard
- Weekly rollup synthesis

### Out of Scope

- Real-time or intra-day logging (interstitial journaling)
- Mobile or web UI for viewing summaries
- Notifications or alerts based on summary content
- Team-level summaries or multi-user aggregation

### Future Considerations

- Monthly/quarterly brag document generation from accumulated dailies
- Pattern detection (recurring blockers, energy trends, collaboration patterns)
- Additional data source integrations

---

## Constraints and Assumptions

### Constraints

- MCP connectors (Gmail, Google Calendar, Slack) only run locally — scheduled task must execute on the user's machine (computer must be awake at scheduled time)
- Claude Code scheduled tasks have tool permission requirements that must be pre-approved via a manual first run
- Summaries are limited to data available through connected tool APIs (no access to private browser history, desktop apps, etc.)

### Assumptions

- The user's computer is typically awake/available at the configured time on scheduled days
- Google Calendar, Gmail, Slack, and Google Drive cover the majority of meaningful daily work activity
- MCP connectors return sufficient data to reconstruct a useful summary without manual input
- GitHub is accessible from the execution environment for git push

---

## Success Criteria

- The system produces a daily summary file for every scheduled day without manual intervention
- Summaries are factually accurate — no hallucinated events or invented details
- Summary format is consistent and parseable by both humans and LLMs
- Files are committed to the repo with correct dates and filenames
- A new Claude Code session can load recent summaries and understand the user's recent work context

---

## Risks and Mitigation

- **Risk:** MCP connector fails silently for one data source, producing incomplete summaries
  - **Likelihood:** Medium
  - **Mitigation:** Completion report lists data sources that returned no results; user reviews weekly

- **Risk:** Computer is off/asleep at scheduled time, task doesn't run
  - **Likelihood:** Medium
  - **Mitigation:** OS scheduled wake policy; or manual "Run now" from sidebar when back online

- **Risk:** Summary quality is too generic or misses key activities
  - **Likelihood:** Medium
  - **Mitigation:** Iterate on skill prompt based on first week of outputs; cross-day context helps

- **Risk:** Git push fails (auth, network, conflicts)
  - **Likelihood:** Low
  - **Mitigation:** Single committer eliminates conflicts; git credentials cached

- **Risk:** Tool permission prompts block unattended execution
  - **Likelihood:** Low (after first run)
  - **Mitigation:** Pre-approve all tools via manual "Run now" before relying on automated schedule

---

**This document was created using BMAD Method v6 - Phase 1 (Analysis)**
