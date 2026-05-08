# Claude Code Skills

A collection of custom Claude Code skills for daily productivity, project management, and operational workflows. These are plain markdown files that install to `~/.claude/commands/` and are invoked with `/skill-name` in Claude Code.

## Installation

```bash
git clone https://github.com/benxmy/claude-skills.git
cd claude-skills

# Install all skills
cp skills/*.md ~/.claude/commands/

# Or install specific skills
cp skills/morning-coffee.md ~/.claude/commands/
cp skills/today.md ~/.claude/commands/
```

## Skills

### Daily Workflow

The core daily loop — plan your day, stay on track, review at close.

| Skill | Description |
|-------|-------------|
| `/morning-coffee` | Daily focus picker. Reads workboard, pulls calendar/email/triage, helps pick 2-3 focuses, flags stalled work |
| `/today` | Quick reference for today's focus plan |
| `/wrap-up` | End-of-day review. Compare plan vs actual, capture carry-forwards, update workboard |
| `/yap` | Activity logger. Narrate what you're doing so Claude builds context over time |

**See also**: [morning-coffee](https://github.com/benxmy/morning-coffee) — a standalone package with setup scripts, provider system, and full documentation for the daily workflow.

### Operational

Chief-of-staff style skills for managing work across projects.

| Skill | Description |
|-------|-------------|
| `/cos` | Cross-project status checks, routing incoming work, follow-up tracking, resource surfacing |
| `/prep` | Prepare for a conversation. Pulls tracking file, Webex DM history, and email threads |
| `/manager-update` | Generate 2-5 topic update for weekly manager 1:1 from workboard, yap log, Webex, email |
| `/note` | Discrete notes for calls/meetings/topics. Tag with people and projects. Search later |
| `/deprioritize` | Park a project and draft stakeholder communications about the deferral |

### Content & Communication

Skills for producing documents, presentations, and written deliverables.

| Skill | Description |
|-------|-------------|
| `/write-customer-doc` | Draft customer-facing docs (feature guides, release notes, KB articles) using content style library |
| `/portfolio-deck` | Generate a portfolio status PPTX deck from workboard data with RAG status |
| `/research` | Structured research framework for product/opportunity or customer/adoption research |

### Integrations

Skills that connect to external tools and services.

| Skill | Description |
|-------|-------------|
| `/aha-sync` | Sync project status between workboard and Aha releases |
| `/webex-recording` | Extract transcript and summary from a Webex recording URL |
| `/slack-mode` | Route notifications and permission prompts to Slack for a set duration |
| `/rotate-hex-token` | Rotate an API token stored in macOS Keychain |

### Personal

Skills for non-work use.

| Skill | Description |
|-------|-------------|
| `/finance` | Personal financial planning — 401k optimization, retirement planning, contribution calculations |
| `/think` | Thinking partner for reflective conversations — career, relationships, strategy |

## Dependencies

Most skills read from a **workboard** (`~/.claude/projects/memory/workboard.md`) and some connect to external services:

| Dependency | Used by | Required? |
|-----------|---------|-----------|
| Workboard file | morning-coffee, today, wrap-up, cos, manager-update, portfolio-deck, deprioritize | Yes (for those skills) |
| duo-msgraph plugin | morning-coffee, prep, manager-update | Optional — graceful skip |
| Webex agent triage | morning-coffee | Optional — graceful skip |
| yap-log.md | wrap-up, manager-update | Optional |
| Chrome (remote debugging) | webex-recording | Required for that skill |
| Aha API key (Keychain) | aha-sync | Required for that skill |
| Slack webhook | slack-mode | Required for that skill |

## Workboard

The workboard is the backbone for most of these skills. See the [morning-coffee repo](https://github.com/benxmy/morning-coffee) for templates, examples, and a guided bootstrapping skill (`/workboard-init`).

## Customization

These are plain markdown files. Edit them directly to change behavior, tone, or workflow. See [morning-coffee/docs/customization.md](https://github.com/benxmy/morning-coffee/blob/main/docs/customization.md) for guidance on common modifications.

## License

MIT
