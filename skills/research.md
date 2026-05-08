---
name: research
description: Structured research framework for product/opportunity research or customer/adoption research. Provides a consistent checklist so you're not making it up as you go.
user-invocable: true
allowed-tools:
  - Read(~/.claude/projects/memory/*)
  - Read(~/projects/content-audit-results/*)
  - Read(~/projects/content-style-library.md)
  - Glob(~/projects/*)
  - Grep(~/projects/*)
  - Grep(~/.claude/projects/memory/*)
  - Bash(~/.config/claude-graph/bin/msgraph email *)
  - Bash(cd ~/Projects/webex-agent:*)
  - Bash(date *)
  - mcp__snowflake__query
  - mcp__snowflake__list_tables
  - mcp__snowflake__list_schemas
  - mcp__snowflake__describe_table
  - mcp__hex__list_projects
  - mcp__hex__get_project_details
  - mcp__hex__run_project
  - Agent
  - Write(~/projects/*)
  - Edit(~/projects/*)
---

# /research — Structured Research Framework

A consistent checklist for the two types of research Ben does most often.
Replaces doing it bespoke every time with a repeatable framework.

Arguments passed: `$ARGUMENTS`

---

## Dispatch

### If `$ARGUMENTS` is empty or `help` — show usage

Display:
```
/research product <topic>      Product/opportunity research (integration, platform, new feature)
/research customer <question>  Customer/adoption research (usage data, segment analysis, adoption rates)
/research status               Show active research projects from MEMORY.md
```

Briefly explain the two modes:
- **Product**: "What should we build and why?" — gathers context from docs, people, and architecture to produce a PRD or project brief.
- **Customer**: "What's actually happening?" — gathers data from Snowflake, Hex, and other sources to answer a specific question about customers or usage.

### If `$ARGUMENTS` starts with `status` — show active research

Read `~/.claude/projects/memory/MEMORY.md` and display the
"Active Research" section. For each item, show what's done and what remains.

### If `$ARGUMENTS` starts with `product` — product/opportunity research

### If `$ARGUMENTS` starts with `customer` — customer/adoption research

---

## Mode 1: Product/Opportunity Research

Use this when exploring a new integration, feature, platform opportunity, or
strategic question. The goal is to go from "we should do something here" to a
clear picture of what, why, who, and how.

### Step 1: Frame the question

Ask Ben (or extract from arguments):
- **What's the opportunity?** One sentence.
- **Who benefits?** (customers, internal teams, partners)
- **What's the hypothesis?** What do we think is true that we need to validate?

Write these down before doing anything else. This prevents scope creep.

### Step 2: Gather existing context

Run these in parallel where possible:

1. **Internal docs** — search for existing work:
   - Grep `~/projects/content-audit-results/` for the topic
   - Check the Identity Everywhere reference (`~/projects/content-audit-results/identity-everywhere/REFERENCE.md`) if it's a Duo/Cisco integration topic
   - Search `~/.claude/projects/memory/notes/` for related meeting notes

2. **Confluence** — check if there's already research or plans:
   - Search Ben's personal space and known project spaces
   - Check `~/.claude/projects/memory/MEMORY.md` for relevant Confluence page IDs

3. **Webex/Email** — find what people have said:
   - Search Outlook for recent threads about the topic
   - If specific people are involved, pull their Webex DM history (use the webex-agent pattern from /prep)

4. **Workboard** — check if this connects to an active project

Present findings as a **Context Summary**:
> Here's what already exists on this topic: [list docs, threads, notes]
> Key people who've been involved: [names]
> Open questions from existing material: [list]

### Step 3: Identify gaps

Compare what you found against what's needed for a complete picture:

| Dimension | Status | Source |
|-----------|--------|--------|
| Customer need / pain point | ? | |
| Market / competitive context | ? | |
| Technical architecture | ? | |
| Dependencies / blockers | ? | |
| Stakeholder alignment | ? | |
| Effort estimate | ? | |
| Success metrics | ? | |

Mark each as: **Have it**, **Partial**, or **Gap**. Focus the remaining
research on the gaps.

### Step 4: Fill gaps

For each gap, suggest a concrete action:
- **Customer need**: "Run a `/research customer` query" or "Talk to [person]"
- **Technical architecture**: "Read [doc]" or "Set up a call with [engineer]"
- **Stakeholder alignment**: "Who needs to agree? Have they been asked?"
- **Competitive context**: "What do competitors offer here?"

Execute what can be done now (reading docs, querying data). Flag what requires
human action (scheduling calls, getting access).

### Step 5: Synthesize

Produce one of these outputs (ask Ben which):

- **Research Summary** — 1-2 page synthesis of findings, gaps, and recommendations
- **Project Brief** — problem, proposed solution, scope, stakeholders, open questions
- **PRD Draft** — full product requirements document (use content style library at `~/projects/content-style-library.md`)

Save output to `~/projects/<project-name>/` (ask Ben for the project directory
or create one).

### Step 6: Update tracking

- Add or update the research entry in MEMORY.md under "Active Research"
- Update the workboard if this connects to an active project
- Note any follow-up actions needed (people to talk to, data to gather)

---

## Mode 2: Customer/Adoption Research

Use this when answering a specific question about customer behavior, adoption,
or usage patterns. The goal is to go from question to answer with data.

### Step 1: Frame the question

Ask Ben (or extract from arguments):
- **What's the question?** Be specific. Not "how's adoption?" but "What % of Premier customers have configured DirSync and are actively syncing?"
- **What decision does this inform?** (prioritization, go-to-market, feature design)
- **What segments matter?** (edition, customer size, geography, time period)

### Step 2: Identify data sources

Check what's available:

| Source | What it has | Access |
|--------|-------------|--------|
| **Snowflake** | Customer data, feature usage, integrations | `mcp__snowflake__query` |
| **Hex** | Existing dashboards and analyses | `mcp__hex__list_projects` |
| **Customer Research Plugin** | Pre-built research flows | `~/Projects/customer-research/` |
| **Confluence** | Existing research, reports | API |
| **Amplitude** | Product analytics (planned) | Not yet connected |

For Snowflake queries:
1. Start with `mcp__snowflake__list_schemas` to orient
2. Use `mcp__snowflake__describe_table` to understand available columns
3. Key tables (from past research):
   - `CUSTOMER_SUMMARY` — customer metadata (filter: `VALID_CUSTOMER`, `TEST_CUSTOMER`, `IS_PAYING`)
   - `DUO_AZURE_CA_INTEGRATIONS` — Azure CA / auth integrations
   - `DUO_AZURE_DIRS` — DirSync data
4. Always filter out test/invalid customers
5. Check `~/.claude/projects/memory/MEMORY.md` Active Research section for past queries and known gotchas

### Step 3: Build the query plan

Before running any queries, write out the plan:

```
Question: <the specific question>
Approach:
  1. <first query — what it answers>
  2. <second query — what it answers>
  3. <join/comparison — how pieces fit together>
Expected output: <table, chart, dashboard>
```

Show this to Ben for confirmation before executing. This prevents wasted
queries and ensures alignment.

### Step 4: Execute queries

Run queries sequentially (each may inform the next):
- Start with a small exploratory query to validate assumptions
- Check row counts and distributions before diving deep
- Note any data quality issues (nulls, unexpected values, join mismatches)
- Save query text and results — you'll need them for the dashboard

**Known gotchas** (from past research):
- CUSTOMER_SUMMARY segment totals can change between snapshots — always note the date
- DirSync syncing is bimodal (active or completely dormant)
- CII-to-CUSTOMER_SUMMARY join has ~30% unmatched rate
- Always use current snapshot, not cached numbers from previous sessions

### Step 5: Analyze and present

Structure findings as:

```
## Research: <Question>
**Date:** YYYY-MM-DD
**Decision this informs:** <what>

### Key Findings
1. <finding> — <number/evidence>
2. <finding> — <number/evidence>

### Data Summary
<table or key metrics>

### Segment Breakdown
<if segments were part of the question>

### Caveats
- <data quality notes, sample size, snapshot date>

### Recommendation
<what to do with this information>
```

### Step 6: Create deliverable

Ask Ben what output he needs:
- **Hex Dashboard** — use `mcp__hex__*` tools to create or update a Hex project
- **Confluence Page** — publish findings
- **Slack/Email Summary** — quick share with stakeholders
- **Just the data** — save to a file for reference

### Step 7: Update tracking

- Add or update the research entry in MEMORY.md under "Active Research"
- Note the snapshot date and key numbers (so future research can compare)
- Update the workboard if this connects to an active project
- Log any follow-up questions that emerged

---

## Principles

These apply to both modes:

1. **Frame before you search.** Write the question down first. Prevents rabbit holes.
2. **Check what exists.** Don't redo work. Past research, existing docs, and prior queries are your starting point.
3. **Show the plan.** Before executing, show Ben what you're going to do. 30 seconds of alignment saves 30 minutes of rework.
4. **Note your sources.** Every finding should trace back to a doc, query, or conversation.
5. **Track what you learn.** Update MEMORY.md so the next research session builds on this one, not from scratch.
6. **Answer the question.** Don't just present data — say what it means and what to do about it.
