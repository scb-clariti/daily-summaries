# STORY-001: Interactive Setup Script

**Epic:** Developer Experience / Onboarding
**Priority:** Must Have
**Story Points:** 5
**Status:** Implemented
**Created:** 2026-03-25
**Sprint:** 1

---

## User Story

As a **new user (technical or non-technical)**
I want to **run a single setup command that asks me questions and configures everything**
So that **I can get daily summaries running without manually editing YAML files or knowing Claude Code internals**

---

## Description

### Background

Setting up Daily Summaries requires configuring user identity, schedule, data sources, creating a private repo for summaries, and setting up Claude Code scheduled tasks. A guided setup wizard replaces manual configuration with an interactive experience.

### Scope

**In scope:**
- Cross-platform setup (works on macOS and Windows)
- Interactive prompts for all configurable values
- Detection of existing `config.yaml` — offer to load/modify or start fresh
- Generates `config.yaml` with validated inputs
- Creates a private GitHub repo for summaries (or uses existing)
- Creates Claude Code scheduled tasks (daily summary + optional weekly rollup)
- Sets up git remote for the private summaries repo
- Commits and pushes the generated config
- Prints a checklist of external prerequisites (Google Drive for Desktop, MCP connectors)
- Validates prerequisites where possible (e.g., check if local Drive path exists)
- Discovers MCP connector UUIDs dynamically via ToolSearch

**Out of scope:**
- Installing Claude Code itself
- Installing Google Drive for Desktop (just checks if it's there)
- Configuring MCP connectors (those are per-account in Claude Code settings)
- GUI / web interface

### User Flow

1. User clones the public repo and runs the setup skill
2. Wizard detects OS (macOS or Windows)
3. Wizard checks for git and GitHub CLI
4. Wizard checks if `config.yaml` already exists
   - If yes: "Found existing config. Load it and modify, or start fresh?"
   - If no: proceed with fresh setup
5. Wizard prompts for:
   - Full name (default: git user.name)
   - Timezone (default: system timezone)
   - Slack user ID (with instructions on how to find it)
   - Google Drive for Desktop local path (auto-detect, validate exists)
   - Which data sources to enable (Calendar, Gmail, Slack, Drive)
   - Format preferences (sections, bullet target)
   - Cross-day context settings
   - Schedule: time and days for daily summary
   - Schedule: enable weekly rollup? Which day?
6. Wizard derives `file_slug` from full name
7. Wizard creates private GitHub repo (or connects to existing)
8. Wizard writes `config.yaml` to the private repo
9. Wizard creates scheduled tasks via Claude Code
10. Wizard commits and pushes config
11. Wizard prints summary and prerequisites checklist

---

## Acceptance Criteria

- [x] Skill runs on macOS and Windows
- [x] Detects existing `config.yaml` and offers load/modify or fresh start
- [x] All configurable fields in `config.yaml` are covered by prompts
- [x] File slug is correctly derived (lowercase, no accents, hyphens for spaces)
- [x] Generated `config.yaml` is valid YAML
- [x] Scheduled tasks are created with user-chosen time and days
- [x] Weekly rollup task is created if user opted in
- [x] Config is committed and pushed
- [x] Google Drive local path is validated
- [x] Prints clear checklist of external prerequisites after setup
- [x] Discovers MCP UUIDs dynamically (no hardcoded account-specific values)

---

## Technical Notes

### Components

- **Setup wizard:** `.claude/skills/setup.md` — Claude Code skill that drives the interactive setup
- **Config file:** `config.yaml` — generated output, stored in private summaries repo
- **Scheduled tasks:** Created via `mcp__scheduled-tasks__create_scheduled_task`

### Implementation Approach

This is a **Claude Code skill** (not a bash script), because:
1. It needs access to `AskUserQuestion` for interactive prompts
2. It needs `Write` / `Edit` to create `config.yaml`
3. It needs `Bash` for git operations and OS detection
4. It needs the scheduled task creation tool
5. It runs cross-platform without needing Python/Node installed

### Slug Derivation Logic

```
Input: "José García"
1. Lowercase: "josé garcía"
2. Remove accents: "jose garcia" (NFD + strip combining chars)
3. Replace spaces with hyphens: "jose-garcia"
4. Keep only [a-z0-9-]: "jose-garcia"
```

### MCP UUID Discovery

Skills use keyword-based ToolSearch (e.g., `gcal list events`, `gmail search`, `slack search`) to discover available MCP tools at runtime, rather than hardcoding account-specific UUIDs.

---

## Dependencies

**Prerequisite:**
- Repository must be cloned and user must be in the repo directory
- Git must be configured
- Claude Code must be installed and authenticated
- GitHub CLI (`gh`) installed for repo creation

**External setup (printed as checklist):**
- Google Drive for Desktop installed and syncing (optional)
- MCP connectors enabled in Claude Code: Google Calendar, Gmail, Slack
- Claude Code permissions approved (first manual run)

---

## Definition of Done

- [x] Skill file `.claude/skills/setup.md` created and tested
- [x] Fresh setup flow works end-to-end
- [x] Config file is valid and complete
- [x] Scheduled tasks created successfully
- [x] Config committed and pushed
- [x] Prerequisites checklist printed
- [x] README references setup wizard

---

**This story was created using BMAD Method v6 - Phase 4 (Implementation Planning)**
