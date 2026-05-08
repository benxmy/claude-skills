---
name: prep
description: Prepare for a conversation with someone. Pulls their tracking file, recent Webex DM history, and Outlook email threads for full context.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/*/memory/meetings/*)
  - Glob(~/.claude/projects/*/memory/meetings/*)
  - Grep(~/.claude/projects/*/memory/notes/*)
  - Read(~/.claude/projects/*/memory/notes/*)
  - Bash(cd ~/Projects/webex-agent:*)
  - Bash(~/.config/claude-graph/bin/msgraph email *)
  - Bash(~/.config/claude-graph/bin/msgraph calendar *)
---

# /prep — Pre-conversation Briefing

Pull context about a person before a call, meeting, or conversation.
Shows their tracking file (topics to raise, past notes), Outlook email history,
and a summary of recent Webex DM history.

Arguments passed: `$ARGUMENTS`

---

## Dispatch

### If `$ARGUMENTS` is empty — show usage

Display:
```
/prep <name>    Get a briefing on a person before a call
```
List available tracking files from `~/.claude/projects/*/memory/meetings/`.

### Otherwise — run the briefing

The argument is a person's name (or partial match).

1. **Find their tracking file.** Glob `~/.claude/projects/*/memory/meetings/*.md`
   and match the filename against the argument (case-insensitive, partial match OK).
   - If no match, say so and list available files.
   - If multiple matches, show them and ask which one.

2. **Read the tracking file.** Display it — this has topics to raise and past discussion notes.

3. **Search for related notes.** Grep `~/.claude/projects/*/memory/notes/` for
   the person's name. If any note files mention them, list them with dates and topics.

4. **Pull today's calendar for meeting context.** Run:
   ```bash
   ~/.config/claude-graph/bin/msgraph calendar 1
   ```
   Check if there's a meeting with this person today. If so, note the time, duration, and any other attendees. This helps frame the prep — "Your 1:1 with Alex is at 2:00 PM (30 min)."

5. **Pull Outlook email history (14 days).** Run:
   ```bash
   ~/.config/claude-graph/bin/msgraph email person "<email_from_tracking_file>" --days 14 --max 20
   ```
   Extract the email from the `**Email:**` line in the tracking file. If no email is in the tracking file, try searching by name:
   ```bash
   ~/.config/claude-graph/bin/msgraph email search "from:<person_name>" --max 10
   ```

   If msgraph commands fail (token expired, not configured), skip gracefully and note it.

6. **Pull Webex DM history (14 days).** Run:
   ```bash
   cd ~/Projects/webex-agent && set -a && source .env && set +a && .venv/bin/python3.12 -c "
   import sys, os, json
   sys.path.insert(0, '.')
   sys.path.insert(0, 'servers')
   from webex_client import WebexClient
   from oauth import get_valid_token
   from datetime import datetime, timedelta, timezone

   token = get_valid_token(os.environ.get('WEBEX_CLIENT_ID',''), os.environ.get('WEBEX_CLIENT_SECRET',''))
   if not token:
       sys.exit(1)
   webex = WebexClient(token)
   email = '<EMAIL_FROM_TRACKING_FILE>'
   after = datetime.now(timezone.utc) - timedelta(days=14)
   spaces = webex.list_spaces(max_results=200, space_type='direct')
   for space in spaces:
       msgs = webex.get_messages(space['id'], max_results=1)
       if msgs and msgs[0].get('personEmail','').lower() == email:
           messages = webex.get_messages(space['id'], after=after, max_results=200)
           lines = []
           for m in reversed(messages):
               sender = m.get('personEmail','?')
               ts = m['created'][:16].replace('T',' ')
               text = m.get('text','[non-text]')
               lines.append(f'[{ts}] {sender}: {text}')
           print(json.dumps({'count': len(messages), 'transcript': chr(10).join(lines)}))
           sys.exit(0)
   print(json.dumps({'count': 0, 'transcript': ''}))
   "
   ```
   Extract the email from the `**Email:**` line in the tracking file.

7. **Summarize all context.** You have up to 3 communication channels. Summarize each, focusing on:
   - Open threads or unresolved questions
   - Things they're waiting on from me (or I'm waiting on from them)
   - Key topics discussed recently
   - Decisions made or pending
   - Context useful for the upcoming conversation

   **Cross-reference** the channels — if an email thread and a Webex thread are about the same topic, connect them. If a topic appears in the tracking file AND in recent comms, note whether it's been discussed or still needs to be raised.

8. **Present the briefing.** Format as:

   ```
   ## Prep: <Person Name>

   ### Meeting Today
   <time, duration, other attendees — or "No meeting scheduled today">

   ### Topics to Raise
   <from tracking file — mark any that appear in recent comms as "discussed" or "in progress">

   ### Past Discussion Notes
   <from tracking file>

   ### Related Notes
   <list of /note files mentioning this person, if any>

   ### Recent Email Context (14 days)
   <your summary of email threads — subjects, key points, open items>
   <message count for reference>

   ### Recent Webex Context (14 days)
   <your summary of the DM history>
   <message count for reference>

   ### Suggested Talking Points
   <synthesize across all sources: what should Ben bring up, ask about, or follow up on?>
   ```

   If no email or Webex history is found for a channel, note that and still show the other sources. The briefing should be useful even with just the tracking file.
