# /today -- Daily Planning & Bandwidth Regulation Skill

A skill that manages your entire workday arc: morning planning, mid-day check-ins, distress regulation, and EOD wrap-up. Uses markdown daily notes as the single source of truth.

## Install (Claude Code)

```bash
claude plugins add aparajita89/claude-today-plugin
```

After installing, run `/today` -- the first-run setup will walk you through configuration interactively.

## Using with other LLMs

The skill is a prompt. You can use it with any LLM that supports system prompts or custom instructions.

**ChatGPT (GPTs)**
1. Create a new GPT
2. Paste the contents of `skills/today/SKILL.md` into the Instructions field
3. Replace `{DAILY_NOTES_DIR}`, `{GOALS_FILE}`, and `{DRIVE_FALLBACK_URL}` with your own values (or remove those sections)
4. Replace all file read/write instructions with "ask the user to paste their daily note" -- GPTs can't access your local filesystem
5. Remove or replace the `date '+%A %H:%M'` shell command with: "Ask the user what day and time it is"

**Cursor / Windsurf / other AI IDEs**
1. Copy the contents of `skills/today/SKILL.md` into your custom rules file (e.g., `.cursorrules`, `.windsurfrules`)
2. These tools have terminal and file access, so the read/write and shell steps work as-is
3. Replace the placeholders with your own paths

**Gemini, Llama, or any LLM with a system prompt**
1. Paste the contents of `skills/today/SKILL.md` as the system prompt
2. Replace all file I/O instructions with "ask the user to paste/provide their daily note contents"
3. Replace the shell command with "ask the user for the current day and time"
4. Replace the placeholders with your own values

### What to strip for non-Claude-Code usage

| Feature | Why it won't work | Replacement |
|---|---|---|
| `date '+%A %H:%M'` shell command | No terminal access | Ask the user for current day/time |
| File reads/writes (`Read`, `Write`, `Edit`) | No filesystem access | Ask user to paste note contents; output updated note for them to copy back |
| First-run setup (self-modifying SKILL.md) | Claude Code specific | Do the placeholder replacement yourself before pasting |
| Google Drive MCP tools | Claude Code specific | Remove, or use the LLM's native integrations if available |

### What works everywhere (the core value)

- Spill counts and carry-forward age tracking
- Zombie task triage (5+ day items forced into triage)
- Energy-aware filtering (low energy = no deep work)
- Ops justification prompts ("why are *you* needed here?")
- Carry-forward cap (max 5, overflow must be triaged)
- Blocked item escalation dates
- Distress regulation script
- Day-of-week energy guidelines
- No-silent-carries rule
- Follow-up response-check dates
- OKR/goal cross-check

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
