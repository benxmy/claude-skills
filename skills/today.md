---
name: today
description: Quick reference for today's focus plan. Shows what you committed to in /morning-coffee.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/memory/daily/*.md)
  - Bash(date *)
---

# /today — Today's Plan

1. Get today's date: `date '+%Y-%m-%d'`
2. Read `~/.claude/projects/memory/daily/YYYY-MM-DD.md` (using today's date).
   - If it doesn't exist, say: "No plan set for today. Run `/morning-coffee` to pick your focus."
3. Display the plan contents exactly as written — short, no commentary.
