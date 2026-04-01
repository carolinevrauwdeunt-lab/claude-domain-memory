# claude-domain-memory

A lightweight knowledge accumulation system for Claude Code. Captures what you learn while working — API quirks, patterns, confirmed rules — and makes it available in every future session automatically.

## The problem it solves

Claude Code has per-project memory, but no built-in way to track patterns that emerge over time. You rediscover the same quirks. Hypotheses never get confirmed or discarded. Knowledge stays in your head instead of the project.

This system adds a `domain` memory type that accumulates structured knowledge across sessions — with a clear lifecycle from observation to hypothesis to confirmed rule.

## How it works

Each project gets one `domain_*.md` file per knowledge area (e.g. `domain_notion_api.md`, `domain_pricing.md`). Each file has three sections:

- **Facts** — confirmed, no ambiguity
- **Hypotheses** — patterns seen but not confirmed; tracked with a confirmation count
- **Rules** — promoted from hypotheses at 5+ confirmations; demoted back if contradicted

At the end of a session, run `/knowledge-update`. Claude reviews what was worked on and updates the relevant domain files — adding facts, incrementing hypothesis counts, and promoting anything that crossed the threshold.

## Installation

### 1. Install the skill

Copy `skills/knowledge-update/` to your Claude Code skills folder:

```bash
cp -r skills/knowledge-update ~/.claude/skills/
```

The `/knowledge-update` command is now available in Claude Code.

### 2. Set up domain files for a project

Copy the template into your project's memory folder:

```bash
# Find your project slug (it mirrors the path with slashes replaced by hyphens)
# e.g. /Users/you/Documents/my-project → -Users-you-Documents-my-project

cp templates/domain_template.md \
  ~/.claude/projects/<project-slug>/memory/domain_<name>.md
```

Edit the frontmatter and rename `<name>` to match the domain (e.g. `notion_api`, `pricing`, `auth`).

### 3. Add it to MEMORY.md

In `~/.claude/projects/<project-slug>/memory/MEMORY.md`, add:

```markdown
## Domain Knowledge
- [Your domain](domain_<name>.md) — one-line description
```

Claude Code auto-loads `MEMORY.md` at session start, so the domain file will be available from then on.

## Usage

At the end of any session where you learned something non-obvious:

```
/knowledge-update
```

Claude will:
1. Identify which domain(s) the session touched
2. Extract facts, new hypotheses, and confirmations
3. Update the relevant `domain_*.md` files
4. Promote any hypothesis that reached 5+ confirmations to a Rule

## What to store

**Good candidates:**
- API quirks not obvious from documentation
- Schema details that caused bugs when assumed wrong
- Patterns confirmed across multiple sessions
- Rules that should apply by default going forward

**Skip:**
- Information already in code comments or build docs
- Things easily found by reading the current files
- One-off task details

## File structure

```
~/.claude/projects/<project-slug>/memory/
  MEMORY.md                  ← index, auto-loaded by Claude Code
  domain_<name>.md           ← one per knowledge domain
  domain_<other>.md
  user_*.md                  ← existing memory types
  feedback_*.md
  project_*.md
  reference_*.md
```

## Example domain file

```markdown
---
name: Notion API domain knowledge
description: Facts and patterns from working with the Notion API
type: domain
domain: notion_api
---

## Facts
- Property names are case-sensitive — "Name" and "name" are different
- Pagination must be a while loop, not recursion — recursion doesn't correctly pass start_cursor

## Hypotheses

| Hypothesis | Confirmations | Last seen |
|---|---|---|
| Filtering by last_edited_time is more reliable than filtering by a Done date field | 3 | 2026-04-01 |

## Rules
_(none yet)_
```

## Domain files are personal

The template and skill are generic. The domain files themselves contain knowledge specific to your projects — they're not meant to be shared. Each person builds their own through use.
