---
name: note
description: Take discrete notes for calls, meetings, or topics. Tag with people and projects. Search and retrieve later.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/*/memory/notes/*)
  - Write(~/.claude/projects/*/memory/notes/*)
  - Edit(~/.claude/projects/*/memory/notes/*)
  - Glob(~/.claude/projects/*/memory/notes/*)
  - Grep(~/.claude/projects/*/memory/notes/*)
  - Bash(date *)
  - Bash(ls *)
  - Bash(touch ~/Projects/telegram-notify/.suppress-stop)
  - Bash(rm -f ~/Projects/telegram-notify/.suppress-stop)
---

# /note — Discrete Note-Taking

Take notes during calls, meetings, or on any topic. Notes are tagged with
people and projects so you can find them later.

**Storage:** `~/.claude/projects/*/memory/notes/`
**Index:** `~/.claude/projects/*/memory/notes/index.md`

Arguments passed: `$ARGUMENTS`

---

## Dispatch

### If `$ARGUMENTS` is empty or `help` — show usage

Display:
```
/note <topic>          Start a new note (will ask for people/projects)
/note add <text>       Append to the current active note
/note list             Show recent notes
/note search <query>   Find notes by person, project, or keyword
/note show <filename>  Display a specific note
/note end              Close the active note session
```

### If `$ARGUMENTS` starts with `search` — find notes

1. Extract the search query (everything after "search").
2. Read `index.md` to check people and project mappings.
3. Also use Grep to search note file contents for the query.
4. Display matching notes with their date, topic, people, and projects.
5. Offer to show any specific note.

### If `$ARGUMENTS` is `list` — show recent notes

1. Read `index.md` and show the "Recent Notes" section (last 15 notes).
2. For each, show: date, topic, people, projects (one line per note).

### If `$ARGUMENTS` is `end` — close active note session

1. Say the note session is closed.
2. Remind the user which note was saved and where.

### If `$ARGUMENTS` starts with `show` — display a note

1. Extract the filename or search term.
2. If it's a partial match, use Glob to find the file.
3. Read and display the note contents.

### If `$ARGUMENTS` starts with `add` — append to active note

1. If there's an active note in this session (you'll know from context),
   append the text to it.
2. If no active note, ask which note to add to (show recent list).

### Otherwise — start a new note

The argument is the topic/title of the note.

1. Get the current date: `date '+%Y-%m-%d'`
2. Ask the user (briefly, in one message):
   - **People**: Who's involved? (names, comma-separated — or skip)
   - **Projects**: Which project(s)? (or skip)
   - **Type**: call, meeting, 1:1, brainstorm, research, or other (default: note)
3. Create the note file. Filename format: `YYYY-MM-DD-<slugified-topic>.md`
   - Slugify: lowercase, spaces to hyphens, remove special chars, max 50 chars.
4. Write the file with this format:

```markdown
# <Topic>
**Date:** YYYY-MM-DD
**Type:** <type>
**People:** <comma-separated names>
**Projects:** <comma-separated projects>

---

## Notes

```

5. Update `index.md`:
   - Add/update entries under "By Person" for each person mentioned.
   - Add/update entries under "By Project" for each project mentioned.
   - Add to "Recent Notes" at the top (keep max 30 entries).

   Format for person/project entries:
   ```
   ### <Person Name>
   - [YYYY-MM-DD — <Topic>](<filename>)
   ```

   Format for recent notes:
   ```
   - **YYYY-MM-DD** — <Topic> | People: <names> | Projects: <projects> | [<filename>](<filename>)
   ```

6. **Suppress Telegram stop notifications** for the active session:
   ```bash
   touch ~/Projects/telegram-notify/.suppress-stop
   ```
7. Tell the user the note is ready. Enter an **active note session**:
   - From this point, treat every user message as content to append to the
     note's "## Notes" section.
   - Respond with a brief acknowledgment each time ("Got it.", "Added.", etc.)
   - If the user says something that's clearly a command or question to you
     (not note content), answer it normally.
   - Continue until the user says **done**, **end**, **/note end**, or starts
     a different task.
   - When ending:
     1. Remove the suppress file: `rm -f ~/Projects/telegram-notify/.suppress-stop`
     2. Confirm the note is saved and show the filename.

---

## Index maintenance

The index at `index.md` is the lookup table. Keep it accurate:

- **By Person**: alphabetical sections, each with a list of linked notes.
- **By Project**: alphabetical sections, each with a list of linked notes.
- **Recent Notes**: reverse chronological, capped at 30. Remove oldest when full.

When updating the index, always read it first, then edit (don't overwrite).

---

## Tips for the user

- You can reference people by first name — the index will group them.
- Projects should match names from the workboard/MEMORY.md when possible.
- Use `/note search alex` to find all notes involving Alex.
- Use `/note search entra lite` to find all notes about the Entra Lite project.
- Notes persist across sessions in your memory directory.
