---
name: code
description: "Implement new features in small worktree slices with Luna xhigh coding agents and Astra high reviews. Use for feature implementation, including a single-slice feature; skip quick fixes already under discussion, documentation-only edits, and investigation."
---

# Feature coding workflow

The lead slices, delegates, verifies, and integrates. Luna implements each feature slice in an isolated worktree; Astra reviews it before the lead commits it. Small slices keep both implementation and review tractable.

The user's instructions take precedence over this workflow. Resolve routine choices from repository evidence, state material assumptions, and continue authorized work. Ask only when a missing decision prevents correct progress; a workflow checkpoint does not itself require permission.

## Model roles

| Role | Model | Reasoning effort |
|------|-------|------------------|
| Implementation and review fixes | `gpt-5.6-luna` | `xhigh` |
| Code review | `gpt-6-astra` | `high` |

The lead delegates implementation and review using the configurations above. Set both values in spawn arguments; a prompt or UI label does not configure the model. Spawn a dedicated Astra reviewer regardless of the lead's model. Assigned coding and review agents perform their own role without spawning further agents.

With `collaboration.spawn_agent`, use these arguments alongside the concrete `message` brief:

```json
{"task_name":"code_slice_1","model":"gpt-5.6-luna","reasoning_effort":"xhigh","fork_turns":"none"}
```

```json
{"task_name":"review_slice_1","model":"gpt-6-astra","reasoning_effort":"high","fork_turns":"none"}
```

Use fresh context and supply the paths, constraints, acceptance criteria, and relevant user decisions explicitly. Full-history forks inherit the parent configuration and do not accept these overrides. Call collaboration tools directly, not through `functions.exec`; on another interface, use its documented model and effort controls. If a requested configuration is unavailable, report the exact limitation and continue independent preparation. Obtain the user's choice before substituting a model.

## 1. Slice before implementation

Check every slice against these criteria before briefing an agent. Split a slice that fails any criterion:

- It changes **one independently testable behavior**. Evaluate the scope, not the sentence's grammar.
- Its expected diff is **at most 400 changed lines across 8 files**, counting additions plus deletions, tests, and new files.
- It has **one primary acceptance command**: a focused test, script, or request. Repository-required checks still apply.
- It is **mergeable on its own**, after declared dependencies: additive, behind a flag, or a pure refactor. The default branch remains green after each merge.

Keep briefs near 15 lines as a diagnostic for scope. Split multiple goals; retain context needed for correctness even when the brief takes more lines.

Prefer these slicing tactics, in order:

1. **Contract first.** Put shared types, schemas, interfaces, migrations, or flags in a tiny slice 0. Dependent work consumes that fixed contract.
2. **Vertical slices.** Prefer one thin route-to-service-to-test path over whole horizontal layers that cannot be verified independently.
3. **Disjoint file ownership.** Move shared edits into a prerequisite or sequence overlapping slices. Two agents must never edit the same file concurrently, even in separate worktrees.

Immediately before implementation, surface the slice table and the assumptions or conflicts resolved while planning. Follow the user's requested deliverable format. Make the plan visible for user steering, then continue under existing authorization; the table does not introduce a new approval requirement.

| Slice | Behavior | Files | Depends on | Verify command | PR |
|-------|----------|-------|------------|----------------|----|

Preserve an existing workstream dependency graph and execute it in bounded waves using the same slice criteria. Use an available dedicated workflow when it fits the task.

## 2. One clean worktree per slice

- Read applicable `AGENTS.md` files and repository instructions before choosing paths or verification commands. Inspect the current checkout, existing worktrees, and running agents; preserve unrelated work.
- Fetch the remote, identify the actual default branch, and record the verified base ref and commit. Use a different base when the user or repository requires it.
- Give each independent slice a fresh branch and sibling worktree from that base, for example:

  ```sh
  git worktree add ../<repo>-<feature>-<n> -b <type>/<feature>-<n> <verified-base-ref>
  ```

- Stack dependent slices on the reviewed, committed parent slice. Record each slice's own base commit so its diff excludes earlier slices. Use an available stack-management workflow when appropriate; ordinary branch-off-branch works otherwise. Record merge order in each dependent PR.
- Do not implement features in the main checkout or reuse a dirty or abandoned worktree blindly. Check uncommitted changes and active agents before reusing a worktree for the same slice.
- Run the required installation or setup in each new worktree. Respect the repository's package manager and lockfiles.
- Leave the worktree in place while its PR is unmerged; its reviewed commits are the recovery point. Remove it after merge only once it is clean and no agent is using it. Do not force removal or discard uncommitted work.

## 3. Delegate implementation to Luna xhigh

One coding agent owns one slice. Give it the absolute worktree path: agents share the filesystem, and spawning does not change their working directory. Run project commands there and limit writes to it; repository instructions and shared references may be read elsewhere.

Use this brief, filling in actual file names and commands:

```text
Goal:        <one sentence, one behavior>
Worktree:    <absolute path for project commands and writes>
Base:        <recorded slice base commit>
In scope:    <explicit files or bounded areas the agent may edit>
Forbidden:   <specific files, other slices, and other worktrees>
Context:     <user decisions, fixed contracts, relevant reference paths>
Rules:       <applicable AGENTS.md paths and repository constraints>
Done means:  <observable outcome, at most 3 bullets>
Verify:      <primary acceptance command; lead runs checks after handoff>
Split, don't stretch: if completion requires another behavior or a file
             outside scope, report the proposed split before expanding.
The lead runs builds, tests, captures, and Git shipping operations.
Report:      implementation ready or blocked; files, outcome, deviations,
             requested checks, artifacts, and remaining work. Do not claim
             verification without an actual result supplied by the lead.
```

- Parallelize independent slices only with disjoint ownership and available agent slots. Sequence overlapping work. The lead uses the time for dependency analysis, integration preparation, or review of another completed slice.
- **One verification owner:** the lead runs builds, tests, and captures serially. A coder can request a check during implementation and receive its output. After handoff, run the acceptance command and required local checks before review; return failures to the coder. "Implementation ready" means ready for verification, not complete.
- Add tests for observable behavior and meaningful failure cases. Avoid tests that merely mirror the implementation. Reuse passing results for unchanged code and environment; repeat or broaden checks only after relevant edits, failures, or unresolved concerns.
- Codex delivers a child's final response to its parent automatically. Use `collaboration.send_message` for blockers or progress while working. Use `collaboration.followup_task` to resume an idle agent with a complete fix-up brief; a plain message does not trigger a new turn.
- Idle or completion status alone is not evidence that the slice is done. Inspect the report, worktree changes, and verification result before deciding whether a follow-up is needed.
- Check growth during implementation with `git status --short`, `git diff --stat`, and `git diff --numstat` against the slice base, accounting separately for untracked files. If an unfinished diff exceeds the slice limits or leaves scope, inspect it and interrupt the agent with `collaboration.interrupt_agent` when needed. Re-slice before it grows further.

## 4. Review with Astra high

Freeze edits to the slice during review. Give a fresh Astra high agent a read-only review brief containing the user requirements, implementation brief, absolute worktree, exact base commit, repository rules, and raw verification artifacts. Include the final diff or instructions for obtaining it, including new files. Supply constraints and evidence without suggesting a verdict.

The reviewer reports actionable defects introduced by the slice, ordered by severity: file/line, reachable failure scenario, supporting evidence, and a concise correction. Separate nonblocking suggestions and unresolved questions from defects. A no-findings result is valid; state remaining validation gaps. Existing unrelated issues and style preferences do not block the slice unless they violate an applicable requirement.

Review requirements:

- **Size first.** Check the full slice diff before reviewing. If it exceeds the slice limits, return it for splitting into independently reviewable slices or stacked PRs. Merely dividing an oversized change into commits within the same oversized PR does not fix the review problem.
- **Read the complete change.** Compare against the recorded slice base, including staged, unstaged, and untracked changes. Check the brief, engineering principles, edge cases, races, error paths, and silent behavior changes that passing tests can miss.
- **Verify claims mechanically.** Use actual diffs and file comparisons for claims such as "verbatim move" or "only these lines changed." Open the referenced screenshots, payloads, or logs; descriptions of artifacts are insufficient.
- **Sweep related behavior.** Before accepting shared-helper changes, enumerate call sites and their behavioral or visual roles. For removals, inspect sibling implementations, display strings, and readers of removed flags. Use `rg` and repository-aware tools to trace the impact.
- **Inspect rendered UI.** For UI changes, compare before/after captures of the affected screens and states. Judge visual weight, animation, empty/error/loading states, and interaction, not just source changes. Capture the baseline before implementation where needed, or reproduce it from the recorded base in isolation.
- **Keep fixes with the owner.** Send actionable findings back to the Luna agent for that slice. Re-review the final diff after fixes; the reviewer does not implement feature changes. Do not edit the slice concurrently with review.

The lead resolves findings using code and evidence, returning confirmed defects to the owner. Review passes when actionable defects are fixed or dismissed with a concrete explanation; agreement between agents is insufficient. Re-review changed code after fixes. If a finding repeats without new evidence, investigate its premise or revise the slice instead of repeating the same fix.

Then complete the repository's full quality gate and relevant end-to-end validation for the final state, reusing still-valid results. Route failures through the same owner-fix-and-review process. Report any unavailable check explicitly; do not call the slice fully verified while required evidence is missing.

Only then commit the reviewed slice and prepare its PR under the user's existing shipping instructions. Use the `done` skill if available and applicable. Keep one PR per slice and describe its behavior, validation, dependencies, and merge order. Honor local-only, draft-only, or no-push instructions; this workflow does not independently authorize publishing or merging.

## 5. Integrate in dependency order

- Merge only when authorized, following the dependency graph and repository conventions.
- After a parent merges, rebase or restack its children onto the updated base before their final review. Update the recorded base, verify that each diff contains only its own slice, and rerun affected validation.
- Bounce conflicts back to the owning slice. The owner of the later slice resolves its rebase conflicts; do not patch another branch to route around them or make downstream work absorb an upstream defect. Resolve upstream defects in their own owning slice.
- Remove merged worktrees only after checking for active agents and uncommitted work. Preserve unmerged recovery points.

## Scope

A feature that fits in one slice still gets one worktree, one Luna xhigh coding agent, and one Astra high review, with one PR when shipping is authorized. When the user explicitly applies this skill to a tiny rename, copy tweak, or config flip, the lead may edit and review directly while retaining worktree isolation. This exception does not apply to a small but substantive feature.
