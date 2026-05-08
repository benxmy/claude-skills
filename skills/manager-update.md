---
name: manager-update
description: Generate a focused 2-5 topic update for a weekly manager 1:1. Pulls from workboard, yap log, Webex, email, and meeting history. Writes approved topics to a shared OneDrive doc.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/-Users-benmyers/memory/*)
  - Glob(~/.claude/projects/-Users-benmyers/memory/meetings/*)
  - Read(~/yap-log.md)
  - Bash(cd /Users/benmyers/Projects/webex-agent:*)
  - Bash(~/.config/claude-graph/bin/msgraph *)
  - Bash(~/projects/.venvs/docx/bin/python *)
  - Bash(curl * graph.microsoft.com *)
  - Bash(date *)
  - Bash(open *)
  - Bash(cat *)
  - Grep(~/.claude/projects/-Users-benmyers/memory/notes/*)
  - Read(~/.claude/projects/-Users-benmyers/memory/notes/*)
  - Edit(~/.claude/projects/-Users-benmyers/memory/meetings/*)
---

# /manager-update — Manager 1:1 Prep

Generate a focused 2-5 topic discussion list for your weekly manager 1:1.
Pulls from workboard, yap log, meeting file, Webex DMs, Outlook emails,
and the previous OneDrive doc entry. Ranks topics by what needs your
manager's input, then writes approved topics to the shared doc.

Arguments passed: `$ARGUMENTS`

---

## Dispatch

### If `$ARGUMENTS` is empty — show usage

Display:
```
/manager-update <name>    Generate weekly manager update
```
List meeting files from `~/.claude/projects/-Users-benmyers/memory/meetings/`
that contain a `**Manager update doc:**` field.

### Otherwise — run the update flow

The argument is a person's name (partial match OK).

1. **Find their meeting file.** Glob `~/.claude/projects/-Users-benmyers/memory/meetings/*.md`
   and match the filename against the argument (case-insensitive, partial match).
   - If no match, say so and list available files.
   - If match found but no `**Manager update doc:**` field, say this person
     isn't configured for manager updates and show how to add the field.

2. **Read the meeting file.** Extract:
   - `**Email:**` — for Webex/email lookups
   - `**Manager update doc:**` — OneDrive filename
   - `## Topics to Raise` — persistent topics to prioritize
   - `## Past Discussion Notes` — context on prior conversations

3. **Get today's date.** Run:
   ```bash
   date '+%Y-%m-%d'
   ```
   Also run:
   ```bash
   date '+%-m/%-d/%Y'
   ```
   Use the short format (e.g., "4/30/2026") for the doc heading.

4. **Gather context from all sources.** Read these in parallel where possible:

   **a. Workboard** — Read `~/.claude/projects/-Users-benmyers/memory/workboard.md`.
   Focus on: items with blockers, items with "Waiting on" populated, items
   whose state has changed recently.

   **b. Yap log** — Read `~/yap-log.md`. Look at the last 7 days of entries.
   Focus on: what projects got time, what was accomplished, what came up
   unexpectedly.

   **c. Previous OneDrive doc entry** — Download and read the most recent
   entry in the shared doc to understand what was discussed last time.
   Avoid repeating items that are resolved. To download:

   i. Check if Graph token is available:
      ```bash
      cat ~/.config/claude-graph/token 2>/dev/null | head -c 20
      ```
      If empty or missing, authenticate:
      ```bash
      ~/.config/claude-graph/bin/msgraph auth
      ```

   ii. Search for the doc:
      ```bash
      ~/.config/claude-graph/bin/msgraph docs search "<Manager update doc value>"
      ```
      Note the URL from the first result matching the exact filename — you'll
      need it later to open the doc in the browser.

   iii. Download via Graph API (URL-encode the filename — replace spaces with `%20`):
      ```bash
      TOKEN=$(cat ~/.config/claude-graph/token) && curl -sL \
        -H "Authorization: Bearer $TOKEN" \
        "https://graph.microsoft.com/v1.0/me/drive/root:/<URL-encoded-filename>:/content" \
        -o /tmp/manager-update-doc.docx
      ```

   iv. Read the most recent entry using python-docx:
      ```bash
      ~/projects/.venvs/docx/bin/python -c "
      from docx import Document
      doc = Document('/tmp/manager-update-doc.docx')
      in_section = False
      lines = []
      for p in doc.paragraphs:
          if p.style.name == 'Heading 1' and not in_section:
              in_section = True
              lines.append(p.text)
              continue
          if p.style.name == 'Heading 1' and in_section:
              break
          if in_section and p.text.strip():
              lines.append(p.text)
      print('\n'.join(lines))
      "
      ```

   **d. Webex DMs (14 days)** — Pull using the webex-agent venv:
      ```bash
      cd /Users/benmyers/Projects/webex-agent && set -a && source .env && set +a && .venv/bin/python3.12 -c "
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
      email = '<EMAIL_FROM_MEETING_FILE>'
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
      Replace `<EMAIL_FROM_MEETING_FILE>` with the actual email from the meeting file.

   **e. Outlook emails (14 days)** — Run:
      ```bash
      ~/.config/claude-graph/bin/msgraph email person "<email>" --days 14 --max 20
      ```

   **f. Related notes** — Grep `~/.claude/projects/-Users-benmyers/memory/notes/`
      for the person's name (case-insensitive). If any note files mention them,
      read the matching files and look for open action items, decisions, or
      context relevant to the upcoming 1:1.

   If any source fails (token expired, API error), skip it gracefully and
   note which sources were unavailable. Continue with remaining sources.

5. **Rank and select 2-5 discussion topics.**

   Review all gathered context and identify topics using this priority:

   - **P1: Blockers/escalations** — things stalled that the manager can
     unblock (e.g., eng resistance, cross-team misalignment, resource gaps)
   - **P2: Decisions** — choices needing their input or sign-off
   - **P3: Wins/milestones** — things shipped, launched, completed, or
     meaningfully progressed
   - **P4: FYI** — context they should have but don't need to act on
     (only include if fewer than 3 topics from P1-P3)

   Selection rules:
   - Always include all P1s
   - Fill remaining slots with P2, then P3, then P4
   - Cap at 5 topics total
   - Minimum 2 topics — if nothing is blocked/pending, surface wins and FYIs
   - Skip anything that was discussed last time and is unchanged
   - Prioritize items from the "Topics to Raise" section of the meeting file
   - If fewer than 2 topics surface, say: "Nothing urgent — consider using
     the time for career development or strategic discussion."

   Each topic gets a tag: `[BLOCKER]`, `[DECISION]`, `[WIN]`, or `[FYI]`.

6. **Present topics for review.**

   Show the topics in this format:

   ```
   ## Manager Update: <Name> (<date>)

   1. [TAG] Topic — 1-2 sentence description focused on what you need
      from the manager or what they should know
   2. [TAG] Topic — description
   3. [TAG] Topic — description

   Approve, edit, or add topics?
   ```

   Wait for user response. The user can:
   - Approve as-is (e.g., "looks good", "approve", "go")
   - Remove topics (e.g., "drop #2")
   - Add topics (e.g., "add: OKR alignment for next quarter")
   - Reword topics (e.g., "reword #1 to focus on the timeline risk")
   - Reorder topics

   If the user makes changes, show the updated list and ask for approval
   again. Repeat until approved.

7. **Write approved topics to the OneDrive doc.**

   a. Read the downloaded doc (`/tmp/manager-update-doc.docx`) with python-docx.

   b. Find the insertion point — the first `Heading 1` paragraph after the
      "Primary Current Projects/Responsibilities" section (or after the
      references list). This is where the previous week's entry starts.

   c. Insert new content before that heading using python-docx:
      - A `Heading 1` paragraph with the date (e.g., "4/30/2026")
      - For each approved topic: a `Normal` style paragraph with the
        numbered topic text.

   d. Strip the `[TAG]` labels from the doc version — those are for the
      review step only, not for the shared doc.

   e. Each topic is 1-2 sentences. Do NOT use sub-bullets, Current State /
      Next Steps / Blockers structure, or project breakdowns. Keep it
      scannable and conversation-ready.

   f. Save the modified doc to `/tmp/manager-update-doc-updated.docx`.

   g. Upload back to OneDrive:
      ```bash
      TOKEN=$(cat ~/.config/claude-graph/token) && curl -s -X PUT \
        -H "Authorization: Bearer $TOKEN" \
        -H "Content-Type: application/vnd.openxmlformats-officedocument.wordprocessingml.document" \
        --data-binary @/tmp/manager-update-doc-updated.docx \
        "https://graph.microsoft.com/v1.0/me/drive/root:/<URL-encoded-filename>:/content"
      ```

   h. If the upload fails with `resourceLocked`:
      - Save the file locally:
        ```bash
        cp /tmp/manager-update-doc-updated.docx ~/Desktop/<filename>
        ```
      - Tell the user: "Doc is locked (probably open in Word). Saved to
        ~/Desktop/<filename> — close the doc and I can retry, or copy it
        manually."

   i. On success, open the doc in the browser:
      ```bash
      open "<doc webUrl from search results>"
      ```

8. **Update the meeting file.**

   - If any "Topics to Raise" items were covered in this update, remove them
     from the meeting file using the Edit tool.
   - If any new follow-up items emerged during the review step, add them
     to "Topics to Raise".

9. **Confirm completion.**

   ```
   Manager update written to "<doc name>" and opened in browser.
   <N> topics for this week's 1:1 with <Name>.
   ```
