---
name: yap
description: Log what you're doing right now — the "Yapper's API". Narrate your activities so Claude builds context over time. Use with no args to review, or 'listen' for ongoing logging mode.
user-invocable: true
allowed-tools:
  - Read(~/yap-log.md)
  - Write(~/yap-log.md)
  - Edit(~/yap-log.md)
  - Bash(date *)
  - Bash(touch ~/notifications/.suppress)
  - Bash(rm -f ~/notifications/.suppress)
---

# /yap — The Yapper's API

A simple activity log. Narrate what you're doing so Claude can track your day,
spot patterns, and build context over time.

## Purpose

The yap log is not just a diary — it's a dataset. The goal is for Claude to
build a rich understanding of how Ben spends his time so we can:

- **Spot patterns**: recurring tasks, context switches, time sinks
- **Identify automation opportunities**: repetitive workflows worth turning
  into skills or scripts
- **Build better processes**: tailor tools and workflows to how Ben actually
  works, not how he thinks he works
- **Surface insights proactively**: when reviewing the log, call out anything
  interesting — not just a summary, but actionable observations

When Ben asks for a `/yap summary` or reviews the log, go beyond listing what
happened. Look for:
- Tasks that keep recurring — could they be automated?
- Time gaps or context-switch patterns that suggest friction
- Projects getting lots of time vs. ones being neglected
- Opportunities to suggest new skills, crons, or workflows

Arguments passed: `$ARGUMENTS`

---

## Dispatch

### If `$ARGUMENTS` is empty — show recent log

1. Read `~/yap-log.md`. If it doesn't exist, say "No yaps yet. Start
   logging with `/yap <what you're doing>`."
2. Show today's entries. If none today, show the most recent day that has
   entries (up to 10 entries).
3. After displaying, offer a brief summary of the day's activity pattern
   (e.g., "Looks like you spent most of the morning on product research, then
   switched to email after lunch.").

### If `$ARGUMENTS` is `summary` — summarize recent activity

1. Read `~/yap-log.md`.
2. Summarize the last 3 days of entries: themes, time allocation, patterns.
3. Keep it concise — bullet points, not paragraphs.

### If `$ARGUMENTS` is `listen` — start a listening session

Enter an ongoing logging mode where every message the user sends gets logged
as a yap entry — no need to prefix with `/yap` each time.

1. Get the current timestamp: `date '+%Y-%m-%d %H:%M'`
2. Check: if it's morning (before noon) and `/morning-coffee` hasn't been run
   yet this session, ask Ben: "Want to run `/morning-coffee` first?"
   If he says yes, run it before entering listen mode. If no, proceed.
3. **Suppress Telegram stop notifications** for the listen session:
   ```bash
   # Optional: suppress notifications during focused logging
   # touch ~/notifications/.suppress
   ```
4. Announce that listening mode is on:
   > "Listening. Just talk — everything you say gets logged. Say **stop** or
   > **/yap stop** when you're done."
3. From this point, treat every subsequent user message as a yap entry:
   - If the message is empty or blank (user just hit enter), do nothing at all
     — no tool calls, no response. Just stay silent and wait for the next
     message. This lets the user "breathe" in the session like a doc.
   - Get a fresh timestamp for each message that has content.
   - Log it to `~/yap-log.md` using the same format as a normal entry.
   - Respond with ONLY a brief acknowledgment (one line max). Vary them
     naturally — "Got it.", "Logged.", "Noted.", etc.
   - Do NOT ask follow-up questions, surface reminders, make suggestions, or
     start conversations. The point of yap is to **listen**, not talk back.
   - The one exception: occasionally you may ask a short follow-up if more
     context would genuinely help you learn something useful about how Ben
     works. Use natural judgment on frequency — not every message, but don't
     be afraid to ask when curious.
   - If the message is a direct question to you, answer it AND log it.
4. Continue until the user says **stop**, **done**, **quit**, **/yap stop**,
   or closes the session.
5. When stopping, show a quick recap: number of entries logged this session
   and a one-line summary of what was covered.

### If `$ARGUMENTS` is `stop` — end listening session

1. If currently in a listening session, end it.
2. Remove the suppress file: `# Optional: re-enable notifications
   # rm -f ~/notifications/.suppress`
3. Show a quick recap: number of entries logged and a one-line summary.
4. If not in a listening session, just say "No active listening session."

### If `$ARGUMENTS` is `clear` — clear the log

1. Confirm with the user before clearing.
2. If confirmed, write an empty file to `~/yap-log.md`.

### Otherwise — log a new entry

1. Get the current timestamp: `date '+%Y-%m-%d %H:%M'`
2. Read `~/yap-log.md` (create if missing).
3. Check if today's date header (`## YYYY-MM-DD`) already exists.
   - If not, append a new date header.
4. Append the entry under today's header:
   ```
   - **HH:MM** — <the user's message>
   ```
5. Write the updated file.
6. Respond with a brief, natural acknowledgment (not robotic). Examples:
   - "Logged."
   - "Got it — deep work on the project."
   - "Noted. Busy morning!"
   Keep it to one short line. Don't parrot back the full entry.

---

## Log file format

`~/yap-log.md`:

```markdown
# Yap Log

## 2026-04-13
- **09:15** — Standup done, starting product research
- **10:30** — Finished first pass on trust level report
- **12:00** — Lunch break

## 2026-04-12
- **08:45** — Morning email triage
- **14:00** — Internal tool debugging session
```

---

## Implementation notes

- Always read before write — don't clobber.
- Keep entries in chronological order within each day.
- Date headers are in reverse chronological order (newest day at top) so
  recent entries are easy to find.
- The log file lives at `~/yap-log.md` — not in the memory directory,
  since it's transient activity data, not persistent knowledge.
- If the log gets very large (500+ lines), mention it and suggest `/yap clear`
  or manual pruning.
