---
name: write-customer-doc
description: Draft customer-facing documents (feature guides, release notes, alpha/beta materials, KB articles) using Ben's content style library. Outputs as markdown, Word, or both.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/*/memory/*)
  - Read(~/projects/content-style-library.md)
  - Read(~/projects/content-audit-results/*)
  - Read(~/Desktop/*)
  - Glob(~/projects/*)
  - Glob(~/Desktop/*)
  - Grep(~/projects/*)
  - Grep(~/.claude/projects/*/memory/*)
  - WebFetch
  - Write(~/projects/*)
  - Edit(~/projects/*)
  - Bash(source ~/projects/.venvs/docx/bin/activate*)
  - Bash(python3 *)
  - Bash(open *)
  - Bash(date *)
  - Bash(ls *)
  - Bash(mkdir *)
  - Agent
---

# /write-customer-doc — Customer-Facing Document Drafting

Draft polished customer-facing documents by pulling from project context, the content
style library, public docs, screenshots, and codebase knowledge.

Arguments passed: `$ARGUMENTS`

---

## Dispatch

### If `$ARGUMENTS` is empty or `help` — show usage

Display:
```
/write-customer-doc feature-guide <topic>   What it does, who sees it, how to use it
/write-customer-doc release-notes <topic>   What shipped and what it means for customers
/write-customer-doc alpha-beta <topic>      Outreach, onboarding, or program materials
/write-customer-doc kb-article <topic>      How-to content, duo.com/docs style
/write-customer-doc <topic>                 Interactive — asks what type you need
```

### If `$ARGUMENTS` has a doc type prefix — extract type and topic

### If `$ARGUMENTS` has no doc type prefix — ask which type

Show the four types with one-line descriptions. Ask the user to pick one.

---

## Step 1: Gather Context

Before asking questions, silently gather what you can. Run these in parallel:

1. **Project memory** — search `~/.claude/projects/*/memory/` for the topic.
   Check project-specific memory files (e.g., `cii-data-in-duo.md`, `entra-lite-cii.md`).

2. **Content style library** — read `~/projects/content-style-library.md`. You will use
   the **professional-direct register** for all customer-facing docs.

3. **Screenshots** — check `~/Desktop/` for recent screenshots (today's date) that may
   show relevant UI. Read any that look related — they're a primary source for feature guides.

4. **Public docs** — if the topic relates to a Duo/CII feature, fetch the relevant page
   from `duo.com/docs` (e.g., `https://duo.com/docs/identity-security`) for accuracy.

5. **Identity Everywhere reference** — if the topic is Duo/CII/identity related, check
   `~/projects/content-audit-results/identity-everywhere/REFERENCE.md`.

Present a brief context summary:
> **Sources found:** [list what you found — memory files, screenshots, docs pages]
> **Gaps:** [anything you couldn't find that you'll need from the user]

---

## Step 2: Clarify

Ask only what you can't figure out from context. One question at a time. Keep it brief.

Things you may need to clarify:
- **Audience** — who reads this? (admins, security teams, procurement, end users)
- **Scope** — what specifically to cover vs. leave out
- **Accuracy** — any details you found that need confirmation
- **Screenshots/visuals** — does the user have additional reference material?

Skip questions you can answer from the gathered context. Don't over-ask.

---

## Step 3: Draft

Write the document using the **professional-direct register** from the content style library:
- Clear, evidence-based, customer-first framing
- Structured with headers and tables
- Warm but not casual
- Em dashes as parenthetical device
- Bold key terms on first use
- Lead with what the customer sees/gets, not internal architecture

### Customer-Facing Writing Lessons (from revision feedback)
- **Include prerequisite/config recommendations up front** — if there's something the customer
  should do to get the most out of a feature, say so in the Overview, not buried later.
  Example: "We strongly recommend adding additional IdP source integrations beyond the default
  Duo integration to maximize accuracy and value."
- **Add alpha/beta disclaimers** — if the doc is for a pre-GA feature, include a NOTE that
  details may change based on feedback. Keep it brief, one sentence.
- **Be specific about role access** — don't list every admin role generically. Only list roles
  that actually have access. Verify before assuming.
- **Link to official docs by name, not "here"** — "See the [Duo Identity Security documentation](...)"
  not "in the documentation here." Links need to survive copy-paste and format changes.
- **Foreshadow upcoming improvements** — when current UX has friction (e.g., complex permission
  setup), briefly mention that a simpler path is coming. Builds confidence without overpromising.
- **Keep the cross-launch pitch concise** — describe what it enables in one sentence. Don't
  list every field in User360. "...enabling your team to seamlessly move from the Duo-specific
  view to the user's full identity context."

### Structure by Doc Type

**feature-guide:**
```
# <Feature Name>
Status / Last Updated metadata

## Overview
What this feature does and why it matters (2-3 sentences, customer value first)

## What You'll See
Walk through the UI — what appears where, what each element means.
Use tables for classifications, status values, etc.

## How to Use It
Task-oriented sections: investigating, configuring, granting access, etc.

## Who Can Access This
Role/permission requirements

## Learn More
Links to full docs, support channels, feedback mechanisms
```

**release-notes:**
```
# <Release Title> — <Date>

## What's New
Bullet list of changes, each with a brief "why it matters"

## Details
Section per feature with screenshots or descriptions

## Getting Started
What to do to take advantage of the new features

## Known Limitations
Honest about what's not included yet
```

**alpha-beta:**
```
# <Program Name>

## What We're Testing
Brief description of the feature and why we need feedback

## What You'll Get
Concrete benefits of participation

## What We Ask
Time commitment, feedback mechanisms

## How to Get Started
Step-by-step onboarding

## Questions?
Contact info, Slack channel, support
```

**kb-article:**
```
# <How to Do X>

## Overview
One paragraph — what this article covers and who it's for

## Prerequisites
What you need before starting

## Steps
Numbered steps with clear actions

## Troubleshooting
Common issues and solutions

## Related Articles
Links to related docs
```

---

## Step 4: Output Format

After drafting, ask:

> **How do you want the output?**
> 1. Markdown file
> 2. Word document (.docx)
> 3. Both

---

## Step 5: Generate and Deliver

### Save location
- Ask the user where to save, or default to `~/projects/<relevant-project-dir>/`
- If the project directory doesn't exist, create it

### Markdown
- Write the file directly

### Word (.docx)
- Generate a Python script that uses `python-docx` to create a styled Word document
- Use the shared venv: `source ~/projects/.venvs/docx/bin/activate`
- Style guidelines for the .docx:
  - Font: Calibri 11pt
  - Use `Light Grid Accent 1` for tables
  - Proper heading hierarchy (Heading 1, 2, 3)
  - Bold inline terms, bullet lists, numbered steps
  - Metadata line (Status, Last Updated) under the title
- Run the script, then clean it up (delete the generation script)

### Open the file
- Run `open` on the output file so the user can review immediately

---

## Principles

1. **Customer-first framing.** Lead with what the customer gets, not how it works internally.
2. **Accuracy over speed.** Check public docs and project memory before making claims about features.
3. **Use screenshots as primary source.** If the user provides screenshots, describe exactly what's shown — don't guess at UI that might have changed.
4. **Professional-direct register.** Match the content style library. Em dashes, bold terms, structured headers, warm but not casual.
5. **Don't over-ask.** Gather context first, then only clarify what you genuinely can't figure out.
6. **Honest about gaps.** If something needs verification, say so rather than guessing.
