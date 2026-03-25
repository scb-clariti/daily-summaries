# Daily Summaries

An automated daily work journal powered by Claude Code. Every day at your chosen time, it gathers your activity from Google Calendar, Gmail, Slack, and Google Drive, produces a structured Markdown summary, and commits it to a **private repository** that only you can see.

---

## How It Works

```
This repo (public)          Your data repo (private, created by setup)
─────────────────           ─────────────────────────────────────────
.claude/skills/       ←──── scheduled task reads skills from here
  daily-work-summary.md
  weekly-rollup.md
  setup.md

.claude/settings.json       config.yaml            ← your settings
                            your-name-daily-summary-YYYY-MM-DD.md
                            weekly-rollup-YYYY-WNN.md
                                    ↑
                        scheduled task writes summaries here
```

Two repos, clean separation:
- **This repo** — skills, settings, and documentation. Safe to share publicly.
- **Your data repo** — your personal config and all generated summaries. Always private.

---

## Quick Start

**Prerequisites:**
- [Claude Code](https://claude.ai/claude-code) installed
- [GitHub CLI](https://cli.github.com/) installed and authenticated (`gh auth login`)
- MCP connectors enabled in Claude Code settings: Google Calendar, Gmail, Slack

**Steps:**

1. Clone this repo:
   ```
   git clone https://github.com/scb-clariti/daily-summaries.git
   cd daily-summaries
   ```
2. Open the folder in **Claude Code**.
3. Run the setup wizard:
   ```
   Run the setup skill
   ```
4. Answer the prompts — the wizard will:
   - Collect your name, timezone, Slack ID, and preferences
   - Create a private GitHub repo for your summaries
   - Discover your MCP connector IDs and grant permissions
   - Run a connectivity test on each data source
   - Create the scheduled tasks
5. Click **"Run now"** in the Scheduled sidebar to run your first summary.

That's it. The system runs automatically from there.

---

## What Gets Generated

**Daily summary** (`your-name-daily-summary-YYYY-MM-DD.md`):
- Summary bullets (meetings, emails, Slack, Drive activity)
- Decisions & Rationale
- Open Loops (with cross-day carry-forward)
- Blockers
- Next Steps
- Transcript source (cleaned narrative)

**Weekly rollup** (`weekly-rollup-YYYY-WNN.md`, every Friday):
- Themes & Focus Areas
- Key Accomplishments
- Aggregated Decisions
- Open loops carried into next week

---

## Repo Structure

```
daily-summaries/              ← this public repo
├── .claude/
│   ├── skills/
│   │   ├── daily-work-summary.md   — skill prompt for daily runs
│   │   ├── weekly-rollup.md        — skill prompt for weekly runs
│   │   └── setup.md                — interactive setup wizard
│   └── settings.json               — generic tool permissions
├── docs/                           — product brief and tech spec
├── .gitignore
└── README.md

your-data-repo/               ← private repo, created by setup
├── config.yaml                     — your personal settings
├── your-name-daily-summary-*.md    — generated daily summaries
└── weekly-rollup-*.md              — generated weekly rollups
```

---

## Reconfiguring

To update your settings (timezone, schedule, sources, etc.):
- Open this skills repo in Claude Code and say: **"Run the setup skill"**
- Or edit `config.yaml` in your data repo directly — the next run will pick it up.

---

## Data Sources

| Source | What it captures |
|--------|-----------------|
| Google Calendar | Meetings attended, organizer vs attendee |
| Gmail | Emails you replied to or forwarded |
| Slack | Messages you sent in channels, DMs, and threads |
| Google Drive | Files modified today (via Google Drive for Desktop local sync) |

All sources are configured per-user and can be individually enabled or disabled in `config.yaml`.

---

## Requirements

- Claude Code with scheduled tasks enabled
- GitHub CLI (`gh`) for repo creation during setup
- MCP connectors: Google Calendar, Gmail, Slack (enable in Claude Code → Settings → MCP Connectors)
- Google Drive for Desktop (optional, for Drive activity tracking)
