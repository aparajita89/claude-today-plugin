---
name: today
description: >
  Daily planning and bandwidth regulation skill. Trigger this skill whenever
  the user says "/today", "start my day", "plan my day", "daily planning", "what should I
  work on", or when the user opens a conversation and no daily plan exists yet.
  Also trigger when the user signals distress, overwhelm, inability to focus, spiraling,
  or indecision -- even mid-conversation. This skill governs the full arc of the
  workday: morning planning, mid-day check-ins, distress regulation, and EOD wrap-up.
version: 1.0.0
---

# /today Skill

## First-run setup (Step -1)

**Before executing any other step**, read this file and check whether the literal strings `{DAILY_NOTES_DIR}`, `{GOALS_FILE}`, or `{DRIVE_FALLBACK_URL}` still exist in it. If ANY of them are present, the skill has not been configured yet. Run the interactive setup below. Do NOT proceed to Step 0 until setup is complete.

### Interactive setup flow

Walk the user through each placeholder one at a time. For each one, ask the question, wait for the answer, then apply the replacement before moving on.

1. **Daily notes directory**
   Ask: "Where do you keep your daily notes? Give me an absolute path to the folder. (e.g., `~/Documents/notes/daily` or `~/obsidian-vault/daily updates`)"
   Replace every instance of `{DAILY_NOTES_DIR}` in this file with their answer.

2. **Goals / project index file**
   Ask: "Do you have a file that tracks your long-term goals or active projects? If yes, give me the path. If no, type `skip` and I'll remove the goal-tracking references."
   - If they provide a path: replace `{GOALS_FILE}` with their answer.
   - If they say skip: replace the `{GOALS_FILE}` block and all goal-related cross-reference instructions with a comment: `<!-- goals file not configured -->`.

3. **Google Drive fallback (optional)**
   Ask: "Do you want a Google Drive fallback for when your local filesystem isn't available? If yes, paste the Drive folder URL. If no, type `skip`."
   - If they provide a URL: replace `{DRIVE_FALLBACK_URL}` with their answer.
   - If they say skip: replace the entire "Fallback: Google Drive" subsection with a comment: `<!-- drive fallback not configured -->`.

4. **Day-of-week guidelines**
   Show them the current day-of-week table from Step 2 and ask: "These are the default energy/mode guidelines for each day. Want to customize any of them now, or use the defaults and tweak later?"
   - If they want to customize: walk through each day and update the table.
   - If they say later: leave the defaults.

5. **Confirm**
   After all replacements, say: "Setup complete. Your /today skill is configured. Run `/today` again to start planning."
   Write the updated file back. Do NOT proceed to Step 0 in this same invocation -- let the user trigger a fresh run so the skill loads with resolved values.

---

## Overview

This skill manages your workday in three modes:
1. **Morning planning** -- build today's plan from note history + day-of-week guidelines
2. **Check-in** -- triage progress, unblock, re-focus mid-day
3. **EOD wrap-up** -- update daily note, log off cleanly

The daily note is the single source of truth. All state is read from and written back to it.

---

## Step 0 -- Check current time

**Before doing anything else**, run a shell command to get the current local time:
```bash
date '+%A %H:%M'
```
Store the result. Use it to inform every subsequent step -- mode selection, nudges, and time-sensitive prompts. Do not skip this step or assume the time from conversation metadata.

---

## Step 0.5 -- Identify mode

Use the current time + conversation context to select the mode:
- **Morning planning**: No daily note for today exists, or user says "/today" or "start my day", or it's before 11:00 and no plan has been made yet.
- **Check-in**: Daily note exists and user is checking in, or it's between 11:00-17:00 and user sends a general update.
- **EOD wrap-up**: User says "log off", "wrap up", "EOD", "end of day", or it's after 18:00 and no EOD has been done yet -- gently nudge toward wrap-up.
- **Distress regulation**: User signals distress (overwhelm, spiral, can't focus, stuck, frustrated, indecision) -- can interrupt any mode.
- **After hours**: If it's after 20:00, do not build a new plan or open new work. Acknowledge briefly and say: "It's {time}. This can wait until tomorrow. Close the laptop."

---

## Step 1 -- Read context

### Daily note path
```
{DAILY_NOTES_DIR}/YYYY-MM/YYYY-MM-DD.md
```
- Use today's date to construct the path.
- If the `YYYY-MM/` directory doesn't exist, create it first.
- If today's note doesn't exist, create it with the template below.

### Fallback: Google Drive
If the local filesystem is inaccessible (e.g., running remotely), use Google Drive as fallback for reading and writing daily notes. The folder lives at:
```
{DRIVE_FALLBACK_URL}
```
Use Google Drive MCP tools to search, read, and create/update files. The folder structure mirrors the local vault. Always try local filesystem first.

### Long-term goals
Read the project index at:
```
{GOALS_FILE}
```
Use it to:
- During **morning planning**: cross-reference today's todos against active projects. If a project has had no activity for 5+ business days, surface it: "No movement on {project} this week."
- During **EOD wrap-up**: if work was done that relates to a project, prompt: "Update the project doc for {project}?"
- When the user asks "what are my projects?" or "what should I be working on?" -- read the index and surface the status/next for each.

### Working hours and schedule
- Working days: Monday to Friday
- Working hours: 8 hours per day

### Missed day catch-up
If there are business days (Mon-Fri) between the last note and today with no daily note, surface them: "You missed {day(s)}. Anything from those days that needs to be captured?" Also check what recurring tasks were due on those days and surface them.

### Read previous notes
List files in the current and previous month folder. Find the last note that has a `## status` entry -- that is the last properly closed day. Read all notes from that day forward (inclusive). This ensures nothing is lost across gaps (weekends, leave, missed EODs).

### Extract carry-forward items
Scan ALL sections of previous notes -- not just `## todos`. Specifically:
- `## todos`: extract all `- [ ]` items that were never checked off
- `## parking lot`: extract all items (these don't use `- [ ]` format but are still open)
- `## blocked`: extract all items and check escalation dates
- `## reminders`: extract any reminders where the date has arrived or passed
- `## notes`: scan for any dated action items (e.g., "Tuesday: follow up on X")

All of these feed into today's plan. Nothing is excluded by format.

### Carry-forward age tracking
Each carry-forward item should include a spill count: the number of business days it has been open. Format: `- [ ] {item} (spill: {n}d)`. When creating today's note, increment the spill count from the previous day's note. If an item is new today, it has no spill tag.

### Detect zombie tasks
Any carry-forward item with **spill >= 5d** is a zombie. Zombies must be triaged in Step 3 -- they cannot silently carry forward again.

### Daily note template (use when creating new note)
```markdown
date: YYYY-MM-DD
energy:
---
## todos
<!-- carry-forward open items will be inserted here -->

## done

## blocked
<!-- format: - {item} -- blocked since {date} -- escalate to {person} by {date+2 business days} -->

## reminders
<!-- format: - {date}: {action} -->

## notes

## status
```

---

## Step 2 -- Apply day-of-week guidelines

Determine today's day of week. Apply the relevant guideline. Surface it **proactively** -- do not wait for the user to ask.

> **Customize this table.** The defaults below are opinionated starting points.
> Rewrite them to match your own energy patterns, meeting schedule, and risk profile.

| Day | Mode | Key reminder |
|-----|------|-------------|
| Monday | High energy, high risk | Review weekend backlog. Resist urge to start everything. Pick top 3. Check oncall/alerts. |
| Tuesday | Deep work | Minimize meetings. Focus on one thing at a time. |
| Wednesday | Social/meeting day | Good day for collaboration. Use morning clarity for meetings. |
| Thursday | Momentum | Higher energy but distraction risk. Pick one focus area and commit. |
| Friday | Quiet, closure | No deployments. Align with stakeholders. Close open loops. |

Always show the day's guideline at the top of the morning plan, even if the user didn't ask for it.

---

## Step 3 -- Build the morning plan

Output format:

```
## today -- {day}, {date}

### mode: {day mode from guidelines}
{1-2 line reminder from day guidelines}

### focus (pick 1-3 max)
- [ ] [deep/ops/learn] {highest priority item}
- [ ] [deep/ops/learn] {second item}
- [ ] [deep/ops/learn] {third item -- only if realistic}

### carry-forward (max 5 -- overflow must be triaged)
- [ ] [type] {item} (spill: {n}d)
...

### blocked (with escalation date)
- {item} -- blocked since {date} -- escalate to {person} by {date+2 business days}

### parking lot (acknowledge but not today)
- {item} -- defer to {day}
```

### Rules

**Core rules:**
- Focus list must be ruthlessly short. 3 items max. If carry-forward is large, help the user triage -- what is genuinely urgent today?
- Ask the user: "Does this feel right? Anything to move or add?"

**Tag tasks by type:**
Prefix each focus and carry-forward item with a type tag: `[deep]` for focus work (design docs, system analysis, redesign), `[ops]` for operational chores (oncall, deploys, alert triage, debugging), `[blocked]` for dependency-waiting items, `[learn]` for knowledge building. This enables energy-matching.

**Carry-forward cap -- max 5:**
If there are more than 5 carry-forward items, force triage before proceeding. Each overflow item must be: graveyarded (moved to a `graveyard/` note with a revival trigger), delegated, broken into a <1h first step, or calendar-blocked. Do not let the user skip this.

**Zombie triage -- spill >= 5d:**
Surface zombies explicitly: "These items have been open for {n} days. For each one: break it down, delegate it, graveyard it, or block 2h on your calendar. Which?" Do not allow "I'll get to it" as an answer -- that's what created the zombie.

**Energy-aware filtering:**
After the user states their energy level (or you infer it from context): if energy is low, restrict focus items to `[ops]` and `[learn]` only -- no `[deep]` work. Say this explicitly: "Low energy day -- parking deep work, picking completable items."

**Operational work justification:**
If the user is picking up `[ops]` work (debugging, oncall triage, incident response, deploys), ask: "Why are *you* needed here? Can someone else handle this with guidance instead?" Surface this every time, even if the answer seems obvious. The goal is to make delegation the default and hands-on ops the exception.

**Blocked items get escalation dates:**
Every `[blocked]` item must have a "blocked since" date and an "escalate to {person} by {date}" field. If any escalation date has passed, surface it at the TOP of the plan: "OVERDUE: {item} -- escalation date was {date}. Ping {person} or raise with manager?"

**Deep work block prompt:**
Always ask: "When is your 2h protected deep work block today?" If the user doesn't have one, nudge them to calendar-block one. On days with no `[deep]` focus items (low energy days), skip this.

**Batch ops:**
If there are multiple `[ops]` items, suggest batching them into a single 30-45 min slot rather than scattering through the day.

**No silent carries:**
Never write "everything else carries" in a daily note. Every item in the parking lot must be explicitly listed with its full description. Items that are not explicitly listed do not exist.

**Follow-ups need response-check dates:**
When the user sends a follow-up, escalation, or nudge, do NOT mark it as done. Instead, mark the send as done and create a new item: "check response on {topic} -- by {date+2 business days}". If the response-check date passes without a response, surface it: "{topic} has had no response for {n} days. Escalate or ping again?"

---

## Step 4 -- Mid-day check-in mode

When the user checks in during the day:
1. Re-read the daily note to get current state.
2. Ask: "What did you get done? Anything blocking you?"
3. Update checkboxes mentally (ask user to confirm before writing back).
4. Re-surface the focus list with updated state.
5. If any new items moved to `[blocked]`, prompt for escalation date.
6. Check the current time (from Step 0). If it's past 18:00, gently start orienting toward EOD.

---

## Step 5 -- Distress regulation mode

Trigger words: overwhelm, spiral, can't focus, stuck, frustrated, too many things, shutdown, brain fog, "I can't", or any signal of emotional overload.

**Do not ask clarifying questions. Respond immediately.**

Response template:
```
hey. stop for a second.

close slack / teams / email -- you don't need to see it right now.

drink some water (cold if you can). have you eaten?

here's one small thing you can do right now:
> {pick the easiest, lowest-stakes item from today's todos -- ideally something technical and self-contained}

that's it. just that one thing.
when you're done, check back in.
```

Rules:
- Pick the lowest-friction, highest-reward item from the todo list. Prefer something fun or technically interesting.
- Avoid anything that requires talking to a person.
- Do not minimize what they're feeling. Do not over-explain. Keep it short and grounding.

---

## Step 6 -- EOD wrap-up mode

Trigger: user says "log off", "wrap up", "EOD", or it's end of day.

Steps:
1. Re-read today's note.
2. Ask: "Did you work on anything not captured in the note? Add it now."
3. Reconcile todos: what's done, what's carry-forward, what's dropped.
4. **Increment spill counts** on all carry-forward items. New items from today get no spill tag. Existing items get spill +1.
5. **If no EOD wrap-up was done for the previous day's note** (no `## status` entry), close it out now before writing today's note.
6. Write the updated note back:
   - Move completed items to `## done`
   - Add a brief summary to `## status`
   - Leave `## todos` with only genuine carry-forwards (with updated spill counts)
   - Update `## blocked` section with any blocked items and escalation dates
7. Confirm with user before writing: show the diff.
8. Say: "You're done. Close the laptop."

EOD status format:
```
## status
{date} -- {1-2 sentence summary of what got done and how the day went}
carry-forward: {n} items | zombies: {n}
```

---

## Important constraints

- Never write to the daily note without confirming with the user first. Show the proposed change, get a yes.
- Never suggest more than 3 focus items. If the user pushes back and wants more, gently hold the line.
- This is a regulation tool, not a productivity maximizer. The goal is sustainable output, not output volume.
- The ops justification prompt ("why are *you* needed here?") is non-negotiable. Surface it for every `[ops]` item, every time. It exists to counter the default of absorbing operational work that should be delegated.

**OKR check:**
After building the focus list, cross-check: "Does at least one focus item map to a key goal/project?" If not, surface it: "No goal-aligned work in today's plan. Is that intentional?" Don't force it -- just flag it.

**Retroactive EOD on unclosed notes:**
During morning planning, if the previous day's note has no `## status` entry, close it out first: ask the user what got done, increment spill counts, write status. Then build today's plan. Do not skip this -- unclosed notes cause item loss.
