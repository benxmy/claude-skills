---
name: portfolio-deck
description: Generate a portfolio status PPTX deck from current workboard data. Covers all active work projects with RAG status, current state, next steps, and blockers.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/memory/*)
  - Read(~/projects/claude-hub/server/*)
  - Bash(cd ~/projects/claude-hub && node *)
  - Bash(open *)
  - Bash(ls *)
---

# /portfolio-deck — Generate Portfolio Status Deck

Generate a PowerPoint deck summarizing all active work (Cisco) projects.

**Output:** `~/projects/portfolio-deck/portfolio-status.pptx`

---

## Process

1. Run the deck generation:

```bash
cd ~/projects/claude-hub && node -e "import('./server/portfolio-deck.js').then(async m => { const p = await m.generateDeck(); console.log('Generated:', p); })"
```

2. Report the result to the user:
   - Number of project slides generated
   - Any projects that had issues (missing data, etc.)
   - The output file path

3. Offer to open the file:

```bash
open ~/projects/portfolio-deck/portfolio-status.pptx
```

## Notes

- Data sourced from `workboard.md`, `okrs.md`, and project memory frontmatter
- RAG status: green (no blockers, active), yellow (waiting on someone or at risk), red (blocked or off track)
- Override RAG per-project by adding `rag: green|yellow|red` to project memory frontmatter
- Only includes "Active — Work (Cisco)" projects
