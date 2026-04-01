---
name: batch
description: Orchestrate large parallelizable changes across a codebase — decomposes work into 5-30 independent units, spawns worker agents in isolated git worktrees, each worker simplifies, tests, commits, and opens a PR.
allowed-tools: Bash, Read, Glob, Grep, Agent, Task
user-invocable: true
---

# Batch: Parallel Work Orchestration

You are orchestrating a large, parallelizable change across this codebase.

## Phase 1: Research and Plan

Enter plan mode, then:

1. **Understand the scope.** Launch subagents to deeply research what this instruction touches. Find all files, patterns, and call sites that need to change. Understand existing conventions so the migration is consistent.

2. **Decompose into independent units.** Break the work into 5–30 self-contained units. Each unit must:
   - Be independently implementable in an isolated git worktree (no shared state with sibling units)
   - Be mergeable on its own without depending on another unit's PR landing first
   - Be roughly uniform in size (split large units, merge trivial ones)

   Scale the count to actual work: few files → closer to 5; hundreds of files → closer to 30. Prefer per-directory or per-module slicing over arbitrary file lists.

3. **Determine the e2e test recipe.** Figure out how a worker can verify its change works end-to-end — not just that unit tests pass. Look for:
   - Browser automation tools (for UI changes: click through the affected flow, screenshot the result)
   - CLI verifiers (for CLI changes: launch the app interactively, exercise the changed behavior)
   - Dev-server + curl pattern (for API changes: start the server, hit the affected endpoints)
   - An existing e2e/integration test suite the worker can run

   If you cannot find a concrete e2e path, ask the user. Offer 2–3 specific options based on what you found. Do not skip this — the workers cannot ask the user themselves.

   Write the recipe as a short, concrete set of steps a worker can execute autonomously.

4. **Write the plan** including:
   - Summary of research findings
   - Numbered list of work units — for each: a short title, the list of files/directories it covers, and a one-line description of the change
   - The e2e test recipe
   - The exact worker instructions you will give each agent

5. Present the plan for approval before proceeding.

## Phase 2: Spawn Workers (After Plan Approval)

After the plan is approved, spawn all worker agents in parallel. Each worker must:

1. **Implement the change** — make exactly the changes described in its unit
2. **Simplify** — use the simplify skill to review and clean up changes
3. **Run unit tests** — check package.json scripts, Makefile targets, or common commands (npm test, bun test, pytest, go test). If tests fail, fix them.
4. **Test end-to-end** — follow the e2e test recipe from the coordinator's plan. If the recipe says to skip e2e for this unit, skip it.
5. **Commit and push** — commit all changes with a clear message, push the branch, and create a PR with `gh pr create`. Use a descriptive title.
6. **Report** — end with a single line: `PR: <url>` so the coordinator can track it. If no PR was created, end with `PR: none — <reason>`.

Use isolated git worktrees for each worker to prevent conflicts. Spawn with `isolation: "worktree"`.

## Phase 3: Aggregate Results

After all workers complete:
- Collect all PR URLs
- Note any failures and their reasons
- Provide a summary of what was done
