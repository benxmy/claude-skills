---
name: webex-recording
description: Extract transcript and summary from a Webex recording. Takes a URL directly or finds one from a recent email. Requires Chrome running with --remote-debugging-port=9222.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - mcp__chrome-devtools__navigate_page
  - mcp__chrome-devtools__list_pages
  - mcp__chrome-devtools__new_page
  - mcp__chrome-devtools__take_snapshot
  - mcp__chrome-devtools__fill
  - mcp__chrome-devtools__click
  - mcp__chrome-devtools__press_key
  - mcp__chrome-devtools__wait_for
---

# /webex-recording — Extract Webex Recording Content

Extract transcript, AI-generated summary, chapters, and action items from a
Webex recording via Chrome DevTools Protocol.

## Prerequisites

Chrome must be running with remote debugging enabled:
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-debug \
  --remote-allow-origins='*'
```

The `chrome-devtools` MCP server must be configured in `~/.mcp.json`.

## Arguments

`$ARGUMENTS` can be:
- A Webex recording URL (e.g., `https://cisco.webex.com/cisco/ldr.php?RCID=...`)
- An email search query to find the recording (e.g., `from:sender weekly demo`)
- Empty — will prompt for URL or email search

## Steps

### 1. Get the recording URL and password

**If `$ARGUMENTS` is a URL:**
- Use it directly
- Ask the user for the password (or check if provided after the URL)

**If `$ARGUMENTS` is an email search query:**
- Search email via: `~/.config/claude-graph/bin/msgraph email search "<query>" --max 1`
- Extract the recording URL from the email HTML body using the Graph API:
  ```bash
  TOKEN=$(cat ~/.config/claude-graph/token)
  curl -s -H "Authorization: Bearer $TOKEN" \
    "https://graph.microsoft.com/v1.0/me/messages/<message-id>?\$select=body"
  ```
- Parse out `href` URLs matching `webex.com` patterns (ldr.php, recordingservice, etc.)
- Extract password from email body text (look for "Password" label followed by text)

**If `$ARGUMENTS` is empty:**
- Ask the user: "Paste a Webex recording URL, or give me an email search query to find one (e.g., 'from:sender weekly demo')"

### 2. Navigate to the recording

1. Check `mcp__chrome-devtools__list_pages` to confirm browser is connected
2. Navigate to the recording URL via `mcp__chrome-devtools__navigate_page`
3. Take a snapshot to check current state

### 3. Enter password (if required)

1. If snapshot shows a password prompt:
   - Find the password textbox element
   - Fill it with `mcp__chrome-devtools__fill`
   - Click OK or press Enter
   - Wait for page to load (take snapshot to confirm)
2. If snapshot shows a login page (SSO):
   - Tell the user: "Please log in manually in the Chrome window, then say 'continue' when you're on the recording page."

### 4. Handle cookie consent

If a cookie banner appears, click "Accept" to dismiss it.

### 5. Extract content

Once on the recording playback page, extract in this order:

**a. Chapters** (from the chapters panel):
- Look for chapter list items in the snapshot
- Extract: chapter number, timestamp, title

**b. Summary** (from the Summary tab):
- Click the Summary tab if not already selected
- Extract: Notes section, Action items section

**c. Transcript** (from the Transcript tab):
- Click the Transcript tab
- The transcript will be in the snapshot as StaticText elements
- Parse into structured format: `[timestamp] Speaker: text`
- Group by speaker turns

### 6. Return structured output

Return the content in this format:

```markdown
# Webex Recording: <title>
**Date**: <extracted from title or URL>
**Duration**: <if available>
**Recording URL**: <url>

## Chapters
1. [0:00] Chapter title
2. [MM:SS] Chapter title
...

## AI Summary
<notes from Webex AI>

## Action Items
<action items from Webex AI>

## Transcript
[0:00] Speaker Name: text
[0:15] Speaker Name: text
...
```

### 7. Save options

After returning the content, ask:
> "Where do you want to save this? Options:
> 1. Note file only (`memory/notes/YYYY-MM-DD-<topic>.md`)
> 2. Note file + update relevant project tracker
> 3. Just show me the content (don't save)
> 4. Custom location"

If the user specified a save location in arguments, use that directly.

## Error Handling

- **Chrome not running**: Tell the user to launch Chrome with the debug port command above
- **MCP not configured**: Tell the user to add `chrome-devtools` to `~/.mcp.json`
- **Password incorrect**: Show error and ask for correct password
- **SSO required**: Prompt user to log in manually
- **Recording not found**: If email search returns no results, ask for a direct URL

## Tips

- For recurring recordings (like weekly demos), combine with a cron job that
  searches for the email and processes automatically
- Transcripts can be very large — for the structured output, consider
  summarizing rather than including the full transcript unless requested
- The Webex AI summary and chapters are usually good enough for a quick overview;
  the full transcript is useful for finding specific quotes or details
