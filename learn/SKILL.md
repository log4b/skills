---
name: learn
description: Capture-and-promote loop for agent workflow lessons. Two modes — capture: at the end of a substantive work session, write a session self-feedback doc (user corrections, friction, gotchas) committed on the work branch; ingest: periodically collect all pending learnings docs, cluster them, and promote durable lessons into hooks, skills, CLAUDE.md, or memory. Use when the user invokes /learn, says "capture what you learned", "session retro", "ingest the learnings", or when wrapping up a work session before the final commit/PR.
---

# Learn — session feedback in, workflow improvements out

Two modes. Resolve which one applies before doing anything:

- Argument `capture` → capture mode. Argument `ingest` (or `review`) → ingest mode.
- No argument: if this conversation contains substantive work to reflect on (code written, a plan iterated, tooling fought), run **capture**. If the session is fresh or the user opened it just for this, run **ingest**.

## Capture mode — write the session's lessons down

You are ending a work session. The PR carries the code; this doc carries what the PR can't: what the user had to correct, where you wasted time, what you discovered the hard way. Write it for the future agent who starts the next session cold.

### What qualifies as a lesson

Only durable, transferable lessons — things that change how the NEXT session behaves:

- **User corrections** — every time the user redirected you, rejected an approach, or restated intent you missed. Record what you did, what they wanted, and the general rule behind it.
- **Friction and wasted work** — steps that took several attempts, tool calls that failed for discoverable reasons, work thrown away. Record the cause and the shortcut.
- **Gotchas discovered** — non-obvious facts about the repo, tooling, or environment that cost time to learn (a flag, a hook behavior, an env quirk).
- **What worked** — an approach or sequence worth repeating deliberately.

Each lesson must be concrete: name the file, command, rule, or message involved. "Be careful with migrations" is not a lesson; "Alembic autogenerate misses X unless Y" is.

Not lessons: task status, what got merged, anything already recorded in the repo's docs or CLAUDE.md (unless the lesson is that the rule didn't fire — say that instead). No secrets, tokens, or patient data — the doc ships in the PR.

**If the session produced no durable lesson, say so and write nothing.** An empty retro committed for ritual is noise.

### The doc

Path: `docs/learnings/YYYY-MM-DD-<branch-slug>.md` if the repo has a `docs/` dir, else `.claude/learnings/YYYY-MM-DD-<branch-slug>.md`. Slug from the branch name (drop ticket prefixes), or a short task slug if no branch.

```markdown
---
date: YYYY-MM-DD
repo: <repo name>
branch: <branch>
pr: <number or none yet>
status: pending
---

# Session learnings — <one-line task description>

## User corrections
- <what I did → what they wanted → the general rule>

## Friction / wasted work
- <what stalled, why, the shortcut for next time>

## Gotchas discovered
- <concrete fact, where it bites>

## What worked
- <approach worth repeating>

## Candidate promotions
- <lesson> → suggested tier: <hook/gate | skill edit (<which skill>) | CLAUDE.md/docs line | memory | none (one-off)>
```

Sections with nothing in them: delete, don't pad. Candidate promotions is your pre-classification for ingest — for each lesson worth keeping, name the most durable tier that could hold it.

### Committing

Commit the doc on the current work branch so it rides the same PR as the work. If the branch is already pushed, push the doc commit too. If there is no work branch (e.g. a research session), commit to wherever the session's output lives, or just write the file and tell the user where it is.

## Ingest mode — turn pending docs into workflow changes

This is an interactive pass. The user approves each promotion; you do the reading, clustering, and editing.

1. **Gather.** Find all learnings docs with `status: pending`: glob `docs/learnings/*.md` and `.claude/learnings/*.md` in the current repo. Ask the user if other repos should be scanned this pass (they may name paths); scan those too. Report the count before proceeding.

2. **Cluster.** Group lessons into themes across docs. A lesson appearing in ≥ 2 sessions outranks a one-off — recurrence is the signal that a promotion pays.

3. **Dedup structurally.** Before proposing anything, check each theme against what already exists: global skills (`~/.claude/skills/`), project skills, `CLAUDE.md` (global and repo), code-standards docs, hooks, memory files. If the rule already exists, the lesson is "the rule didn't fire" — propose better enforcement or placement (move it up a tier, put it where the agent actually reads it), never a second copy of the prose. Restated copies drift.

4. **Propose, then apply.** For each theme, propose the most durable tier that can hold it, with the concrete edit. Get the user's approval (batch related ones; use AskUserQuestion for real forks). The ladder, most durable first:
   1. **Deterministic hook or gate script** — for mechanical rules; a gate kills a mistake class permanently, a prompt line only probabilistically.
   2. **Skill edit** — when the lesson corrects a workflow a specific skill owns, fix that skill's instructions.
   3. **CLAUDE.md / code-standards line** — repo-level rules agents must always see.
   4. **Memory file** — personal preferences and cross-repo context that don't belong in any repo.
   5. **No promotion** — genuine one-offs; record the disposition and move on.

   Routing: team-shareable + repo-specific → repo assets, committed on a branch as a normal PR. Personal or cross-repo → global skills and memory, edited directly.

5. **Close the loop.** Flip each processed doc's frontmatter to `status: ingested`, add `ingested: YYYY-MM-DD`, and append a one-line disposition per lesson (promoted → where / dropped → why). Commit the flips in the repos they live in. Finish with a short tally: N lessons — X promoted (by tier), Y already-covered, Z dropped.

Rules for the pass:

- Never promote a lesson you can't state as a concrete behavioral instruction. Vague lessons stay in the doc, unpromoted.
- Prefer editing an existing asset over creating a new one; prefer one strong line over a paragraph.
- If a repo-destined promotion overlaps a repo's own improvement machinery (e.g. pygeia's promotion pass), offer to hand it there instead of applying directly.
