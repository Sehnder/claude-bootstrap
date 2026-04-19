> **Note:** This is the public bootstrap mirror of this file.
> Canonical copy: `projects/general/skills/SKILL-milestone-logging.md` in the private `claude-knowledge` repo.
> If these differ, the private copy is authoritative.

# Milestone Logging — SKILL.md

## About this file
Defines rules and procedures for maintaining a session log during Cowork sessions.
Load at session startup alongside SKILL-github-knowledge-sync.md.

Last updated: 2026-04-18

---

## Trigger Phrases

Log a milestone when the user says any of the following (or similar):
- "log progress", "log milestone", "checkpoint", "save progress"
- "push our progress" / "push to git" → triggers PUSH (implies LOG first if new content exists)
- "closeout" / "end session" → triggers CLOSEOUT

Claude should also auto-log at natural breaks when:
- A discrete task completes (file created, repo updated, config changed)
- The direction of the session changes significantly
- A significant finding or gotcha is discovered

**Timing rule:** Never interrupt mid-task. Auto-log at the natural break after
completing something, during the response that follows — not as a separate step.

**Skip logging:** If the user says "skip logging until I say resume", suppress
auto-logging until explicitly told to resume. Manual triggers ("log progress")
still work during a skip.

---

## File Locations

**Local (workspace):**
```
{workspace}/session-logs/session-log-YYYY-MM-DD.md
```
- `{workspace}` = the mounted workspace folder (e.g., `Cowork Basics`)
- Full path example: `/sessions/.../mnt/Cowork Basics/session-logs/session-log-2026-04-18.md`
- Multiple sessions same day: `session-log-YYYY-MM-DD-2.md`, `-3.md`, etc.

**GitHub (claude-knowledge):**
```
session-logs/{workspace-name}/YYYY-MM-DD.md
```
- Multiple sessions same day append to the same GitHub file (date is shared)

---

## Entry Format

```markdown
### Milestone — [brief topic] — HH:MM

**Actions:** what was completed since last milestone
**Decisions:** choices made and why
**Findings:** what worked, what didn't, gotchas worth keeping
**Open:** unresolved items or next steps
```

Get current time by running: `date +%H:%M` via Bash tool.

---

## LOG Procedure

1. Run `date +%H:%M` to get current time
2. Read the local session log file (if it exists)
3. If file doesn't exist: create it with the session header (see First Run)
4. Synthesize what has happened since the last milestone entry (or since session
   start if no entries yet) — use current conversation context, not transcript tool
5. Append the new milestone entry
6. Write the updated file

**LOG is local only. Nothing goes to GitHub.**

---

## First Run (no session log file exists)

Create the file with this header before appending the first milestone entry:

```markdown
# Session Log — YYYY-MM-DD

**Workspace:** [name]
**Goal:** [one-line description of session goal]
**Status:** in-progress

---
```

---

## PUSH Procedure

1. Read the local session log file
2. Find the last `---PUSHED YYYY-MM-DD HH:MM---` marker (if any)
3. Extract all content after that marker (the unpushed block)
4. **If nothing new since last marker: skip silently** — no GitHub write, no new marker
5. If new content exists:
   a. Fetch `session-logs/{workspace}/YYYY-MM-DD.md` from claude-knowledge (get SHA if exists)
   b. Append the unpushed block to remote content (or create file if it doesn't exist)
   c. Write back using the push pattern from SKILL-github-knowledge-sync.md
   d. Append push marker to local file: `---PUSHED YYYY-MM-DD HH:MM---`

---

## CLOSEOUT Procedure

1. Run LOG — final milestone entry summarizing session end and overall status
2. Update the Status line in the file header: `**Status:** complete`
   (or `interrupted` if the session is ending before the goal is reached)
3. Run PUSH
4. Done — the GitHub log is now up to date

---

## Resuming an Interrupted Session

When starting a new session after an interruption:
1. Read the local session log file (check `session-logs/` in the workspace)
2. If no local file: fetch the latest log from `session-logs/{workspace}/` in claude-knowledge
3. Review Open Items from the last milestone entry
4. Brief the user on where things were left off before proceeding

---

## Known Edge Cases

| Situation | Behavior |
|-----------|----------|
| GitHub file doesn't exist yet | Create it (omit SHA in API call per sync skill) |
| Nothing new since last push marker | Skip silently |
| Multiple sessions same day | New local file (`-2.md`), append to same GitHub file |
| Session interrupted before closeout | Status stays `in-progress`; next session picks it up |
| Auto-log suppressed (skip mode) | Manual triggers still work; auto-triggers are held |
