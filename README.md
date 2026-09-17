# skills

Agent skills, installable into any coding agent with the [`skills`](https://www.npmjs.com/package/skills) CLI.

## Install

```bash
# Install all skills, pick agents interactively
npx skills add log4b/skills

# Install one skill
npx skills add log4b/skills --skill code

# Install globally (~/.claude/skills) for Claude Code, no prompts
npx skills add log4b/skills --skill '*' -g -a claude-code -y
```

List before installing, update, or remove:

```bash
npx skills add log4b/skills --list
npx skills update code learn
npx skills remove code
```

Run a skill once without installing it:

```bash
npx skills use log4b/skills@code | claude
```

## Skills

| Skill | Description |
|-------|-------------|
| [`code`](skills/code/SKILL.md) | Feature implementation in small worktree slices — coding agents implement, a reviewer agent reviews, the lead integrates. |
| [`learn`](skills/learn/SKILL.md) | Capture-and-promote loop for agent workflow lessons: capture session self-feedback, later ingest it into hooks, skills, or memory. |

## Adding a skill

Each skill is a directory under `skills/` containing a `SKILL.md` with YAML frontmatter:

```markdown
---
name: my-skill
description: "What it does and when to use it."
---

# My skill

Instructions for the agent.
```

Two rules keep skills discoverable by the CLI:

- Live under `skills/<name>/` (up to two category levels are also walked, e.g. `skills/<category>/<name>/`). Outside these paths the CLI falls back to a recursive scan, which is slower and picks up stray files.
- **Always quote the `description`.** Unquoted values containing `: ` (a colon followed by a space) are a YAML nested-mapping error, and the CLI silently skips the skill.

Verify discovery before pushing:

```bash
npx skills add ./ --list
```

Every skill you added should be listed and no `Skipped …` warning should appear.
