---
name: aha-sync
description: Pull Aha release data, compare with workboard status, and interactively push updates. Helps keep Aha tickets current without manual editing.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/memory/*)
  - Write(~/.claude/projects/memory/aha-cache.json)
  - Read(~/projects/claude-hub/server/*)
  - Bash(cd ~/projects/claude-hub && node *)
  - Bash(security find-generic-password *)
  - Bash(security add-generic-password *)
  - Bash(curl * <your-aha-instance>.aha.io *)
  - Bash(date *)
---

# /aha-sync — Aha Release Sync

Synchronize project status between your workboard and Aha releases.

Arguments: `$ARGUMENTS`

---

## Setup Check

First, verify Aha API key is configured:

```bash
security find-generic-password -s "aha-api-key" -w 2>/dev/null && echo "KEY_FOUND" || echo "NO_KEY"
```

If `NO_KEY`:
1. Ask the user to generate an API key at `https://<your-aha-instance>.aha.io/settings/api_keys`
2. Once they provide it, store it:
```bash
security add-generic-password -s "aha-api-key" -a "$USER" -w "THE_KEY" -U
```
3. Then proceed with the sync.

---

## Pull Mode (default, or `$ARGUMENTS` = "pull")

1. Pull latest data from Aha for all linked projects:

```bash
cd ~/projects/claude-hub && node -e "
import { discoverProjects } from './server/discovery.js';
import { pullAllLinked, readCache } from './server/aha.js';
const projects = discoverProjects().filter(p => p.category === 'work');
const cache = await pullAllLinked(projects);
console.log(JSON.stringify(cache, null, 2));
"
```

2. Display a comparison table to the user:
   - Project name | Workboard Status | Aha Status | Aha Confidence | Days Since Sync | Stale?

3. Flag mismatches and stale items (7+ days since last sync or workboard/Aha status diverged).

4. If no linked projects exist, show which work projects are available and suggest linking them by adding `aha: RELEASE-ID` to their memory file frontmatter.

---

## Push Mode (`$ARGUMENTS` = "push" or when user confirms updates)

For each flagged/stale item:

1. Show the user the current workboard state (status, next steps, blockers)
2. Propose updated Product Confidence (Green/Yellow/Red) based on RAG logic:
   - Green: no blockers, active with next steps
   - Yellow: waiting on someone or KR confidence "At Risk"
   - Red: blocked or KR confidence "Off Track"
3. Propose updated Product Confidence Notes (2-3 sentences summarizing current state)
4. Ask user to confirm or edit before pushing

Push via:
```bash
cd ~/projects/claude-hub && node -e "
import { pushUpdate } from './server/aha.js';
const result = await pushUpdate('RELEASE_ID', {
  productConfidence: 'VALUE',
  productConfidenceNotes: 'NOTES'
});
console.log(JSON.stringify(result));
"
```

---

## Linking a New Project

If the user wants to link a project that's currently unlinked:

1. Search Aha for the release:
```bash
curl -s -H 'Authorization: Bearer KEY' 'https://<your-aha-instance>.aha.io/api/v1/releases?q=SEARCH_TERM' | python3 -c "import sys,json; data=json.load(sys.stdin); [print(f'{r[\"id\"]}: {r[\"name\"]}') for r in data.get('releases',[])]"
```
2. Once identified, tell user to add `aha: RELEASE-ID` to the project's memory file frontmatter
3. Then pull fresh data for the newly linked project

---

## Notes

- Aha API base: `https://<your-aha-instance>.aha.io/api/v1/`
- Auth: Bearer token from macOS Keychain (service: `aha-api-key`)
- Cache file: `~/.claude/projects/memory/aha-cache.json`
- Custom fields may need adjustment once we see the actual Aha API response structure
