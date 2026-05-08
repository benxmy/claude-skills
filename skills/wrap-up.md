---
name: wrap-up
description: End-of-day review. Compare what you planned vs what happened, capture carry-forwards, and update the workboard.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/memory/daily/*.md)
  - Read(~/.claude/projects/memory/workboard.md)
  - Read(~/.claude/projects/memory/MEMORY.md)
  - Read(~/yap-log.md)
  - Edit(~/.claude/projects/memory/workboard.md)
  - Edit(~/.claude/projects/memory/MEMORY.md)
  - Edit(~/.claude/projects/memory/daily/*.md)
  - Bash(date *)
---

# /wrap-up — End-of-Day Review

Arguments passed: `$ARGUMENTS`

## Steps

1. **Get today's date**: `date '+%Y-%m-%d (%A)'`

2. **Read today's plan** from `~/.claude/projects/memory/daily/YYYY-MM-DD.md` (using today's date).
   Also read `~/yap-log.md` for any activity entries from today — this is additional context for what actually happened, not a separate review step.
   - If no plan exists or the date doesn't match today, say: "No plan was set for today. Want to do a freeform review instead?" Then skip to step 4.

3. **Show the plan and ask what happened.** Display each focus item and ask Ben to mark them:

   > Here's what you planned today:
   >
   > 1. **Item** — done / partial / didn't get to it
   > 2. **Item** — done / partial / didn't get to it
   > ...
   >
   > How'd it go?

   Let Ben respond naturally — he might say "got 1 and 3 done, didn't touch 2" or give more detail. Don't force a rigid format.

4. **Capture anything else.** Ask briefly:
   > "Anything else notable today that wasn't on the plan?"

   This catches ad-hoc work, decisions made, or things that came up. Keep it quick — one sentence per item is fine.

5. **Identify carry-forwards.** For anything not done or partially done:
   - Ask if it should carry forward to tomorrow or if priorities changed
   - If it's been carrying forward multiple days, flag it: "This has been on the list for a few days. Still the right priority, or should we park it?"

6. **Update the workboard.** Based on what Ben shares:
   - Update "Next step" for items that progressed
   - Update "State" for items that changed
   - Add new blockers or waiting-on items if mentioned
   - Move anything to Parked if Ben decides to defer

7. **Update MEMORY.md reminders** if needed:
   - Add carry-forwards as specific reminders for the next workday
   - Remove any specific reminders that were completed today

8. **Append a summary to today's daily file** so there's a record:
   ```
   ## Wrap-up

   - **Done**: item, item
   - **Partial**: item — what's left
   - **Carry-forward**: item — reason
   - **Unplanned**: anything notable that came up
   ```

9. **Close it out.** One line — "Good day" or "Tough day" or whatever fits. Don't overdo it. If Ben got his writing time, acknowledge it.

## Tone

Same as /morning-coffee — direct, practical, no filler. This should take 2-3 minutes, not 10. Don't ask too many questions. If Ben gives short answers, that's fine — capture the essentials and move on.

## If `$ARGUMENTS` contains a message

Treat it as Ben telling you how the day went. Read the plan, match his response to the items, do the updates, and confirm. Skip the interactive questions.
