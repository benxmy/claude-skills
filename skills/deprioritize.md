---
name: deprioritize
description: Park a project and draft stakeholder communications about the deferral. Helps you consciously deprioritize with clear messaging.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/memory/workboard.md)
  - Edit(~/.claude/projects/memory/workboard.md)
  - Bash(date *)
---

# /deprioritize — Consciously Park Work

Arguments passed: `$ARGUMENTS`

## Purpose

Deprioritizing is a skill. The hard part isn't deciding — it's communicating clearly so stakeholders aren't surprised and you don't carry guilt about it. This skill helps with both.

## Steps

### 1. Identify what to park

If `$ARGUMENTS` names a project, use that. Otherwise, read the workboard and ask Ben which item(s) he wants to park.

### 2. Gather context

Ask (briefly — don't make this a questionnaire):
- **Why now?** What changed, or what's more important? (One sentence is fine.)
- **Who needs to know?** Manager, teammates, stakeholders? List names if possible.
- **When might this resume?** "After X ships", "next quarter", "probably never" — all valid.
- **Is there a handoff?** Does someone else need to pick this up, or can it just wait?

### 3. Update the workboard

Move the item from Active to **Parked** with this format:

```
### [Project Name]
- **Parked**: YYYY-MM-DD
- **Reason**: [one line]
- **Resume when**: [condition or "indefinite"]
- **Communicated to**: [names, or "no one yet"]
```

### 4. Draft the communication

Based on who needs to know, draft a message. Match the channel:

- **Slack/Webex message** — casual, 2-3 sentences. Lead with what you're prioritizing instead, not what you're dropping.
- **Email** — slightly more formal. Include: what's paused, why, what you're focusing on instead, when you expect to revisit.
- **Manager 1:1 talking point** — a bullet point they can put in their notes.

**Tone guidelines:**
- Frame as a prioritization decision, not an apology
- Lead with what you ARE doing (the higher-priority thing), then mention what's moving to the back burner
- Be specific about timing: "parking until after X" is better than "deprioritizing for now"
- If relevant, mention what's already been accomplished before parking

Present the draft(s) and let Ben edit before sending anything.

### 5. Confirm

Summarize: what was parked, who was (or will be) notified, and what the resume condition is.

## If no arguments

Read the workboard and ask: "What are you thinking about parking?" Show the active items list for reference.
