# /today -- Daily Planning Skill for Claude Code

A Claude Code plugin that manages your entire workday arc: morning planning, mid-day check-ins, distress regulation, and EOD wrap-up.

It uses markdown daily notes as the single source of truth and enforces sustainable work habits through carry-forward tracking, zombie task triage, energy-aware filtering, and operational work delegation prompts.

## Install

```bash
claude plugins add aparajita89/claude-today-plugin
```

## Setup

After installing, open `skills/today/SKILL.md` and replace these placeholders with your own values:

| Placeholder | Description | Example |
|---|---|---|
| `{DAILY_NOTES_DIR}` | Absolute path to your daily notes folder | `~/notes/daily` |
| `{GOALS_FILE}` | Path to your long-term goals / project index | `~/notes/projects/goals.md` |
| `{DRIVE_FALLBACK_URL}` | (Optional) Google Drive folder for cloud fallback | `https://drive.google.com/drive/...` |

You should also customize:
- **Day-of-week guidelines** (Step 2) -- rewrite the energy/mode table to match your own rhythms
- **Distress regulation script** (Step 5) -- adjust the grounding response to what works for you

## Usage

- `/today` -- start morning planning
- "start my day", "plan my day", "what should I work on" -- also triggers morning mode
- Check in mid-day and it switches to check-in mode
- "wrap up", "EOD", "log off" -- triggers end-of-day wrap-up
- Signals of overwhelm, frustration, or inability to focus trigger distress regulation immediately

## Key features

- **Spill counts**: tracks how many business days each carry-forward item has been open
- **Zombie triage**: items open 5+ days are forced into triage -- no silent carries
- **Energy filtering**: low energy days automatically park deep work
- **Ops justification**: every operational task gets a "why are *you* needed here?" prompt
- **Escalation dates**: blocked items require a person and deadline for escalation
- **Carry-forward cap**: max 5 items -- overflow must be graveyarded, delegated, or broken down
- **No silent carries**: every item must be explicitly listed or it doesn't exist

## Daily note format

```markdown
date: YYYY-MM-DD
energy:
---
## todos
## done
## blocked
## reminders
## notes
## status
```

## License

MIT
