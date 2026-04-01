# claude-code-skills

A collection of Claude Code skills inspired by design patterns observed in production AI coding assistants. Each skill is a markdown file — drop it into any project and invoke it as a slash command.

## Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| [verify](.claude/skills/verify.md) | `/verify` | Adversarial verification — tries to break your implementation, not confirm it |
| [simplify](.claude/skills/simplify.md) | `/simplify` | 3 parallel code review agents: reuse, quality, efficiency |
| [batch](.claude/skills/batch.md) | `/batch` | Orchestrate large parallelizable changes in isolated git worktrees |
| [security-review](.claude/skills/security-review.md) | `/security-review` | 3-phase security review with false-positive filtering (>80% confidence threshold) |
| [pr-comments](.claude/skills/pr-comments.md) | `/pr-comments` | Fetch and display all comments from the current branch's GitHub PR |
| [skillify](.claude/skills/skillify.md) | `/skillify` | Capture this session's repeatable process as a reusable skill file |
| [explore](.claude/skills/explore.md) | `/explore` | Deep read-only codebase exploration — parallel, structured report |
| [plan](.claude/skills/plan.md) | `/plan` | Software architect agent — design an implementation plan before writing code |
| [stuck](.claude/skills/stuck.md) | `/stuck` | Diagnose frozen/slow Claude Code sessions — CPU, memory, process state |
| [remember](.claude/skills/remember.md) | `/remember` | Review auto-memory and propose promotions to CLAUDE.md / CLAUDE.local.md |

## Installation

### One-liner (all skills)

```bash
mkdir -p ~/.claude/skills && for skill in verify simplify batch security-review pr-comments skillify explore plan stuck remember; do curl -s "https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/$skill.md" -o ~/.claude/skills/$skill.md; done
```

### Individual skills

```bash
mkdir -p ~/.claude/skills
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/verify.md -o ~/.claude/skills/verify.md
# repeat for other skills
```

### Project-level (one project only)

```bash
mkdir -p .claude/skills
# curl skills into .claude/skills/ instead of ~/.claude/skills/
```

Restart Claude Code after installing.

## Skill Details

### `/verify` — Adversarial Verification

Standard verification asks: *does this work?* Adversarial verification asks: *how would this break?*

The key insight: the implementer already has confirmation bias — they built it, tested the happy path, and believe it works. A verification agent that shares that bias adds no value.

Two failure modes this skill is designed to prevent:
- **Verification Avoidance** — reading code and declaring PASS without running anything
- **Fooled by First 80%** — happy path passes, so you stop

Every check must show exact command + exact output. Output is always:
```
VERDICT: PASS | FAIL | PARTIAL
```

---

### `/simplify` — Code Review (3 Parallel Agents)

Launches three specialist agents in parallel, each with a different lens:
1. **Code Reuse** — finds existing utilities that replace newly written code
2. **Code Quality** — catches redundant state, copy-paste, stringly-typed patterns, unnecessary comments
3. **Efficiency** — catches N+1 queries, missed concurrency, hot-path bloat, memory leaks

All three run concurrently and report back. Issues are fixed after all findings are aggregated.

---

### `/batch` — Parallel Work Orchestration

For large refactors that can be decomposed into independent units:
1. Decomposes work into 5–30 independent units
2. Spawns worker agents in isolated git worktrees
3. Each worker implements, tests, commits, and opens a PR
4. Coordinator aggregates all PR URLs

---

### `/security-review` — Security Review

3-phase methodology designed to minimize false positives:
1. Repository context research
2. Comparative analysis against existing patterns
3. Vulnerability assessment

Only reports HIGH/MEDIUM findings with >80% confidence. 14 hard exclusions (DoS, rate limiting, outdated deps, etc.) to avoid noise.

---

### `/pr-comments` — PR Comments

Fetches both PR-level and inline code review comments from the current branch's GitHub PR via the `gh` CLI. Displays them with diff context so you can action them without leaving the terminal.

---

### `/skillify` — Session to Skill

Interviews you about the current session's repeatable process, then writes a `.claude/skills/<name>.md` file you can invoke later. Turns one-off workflows into reusable slash commands.

---

### `/explore` — Codebase Exploration

Read-only, fully parallelized codebase exploration. Returns a structured report:
- Architecture and data flow
- Relevant files for the task
- Key patterns and conventions
- Potential gotchas
- Suggested entry points

---

### `/plan` — Implementation Planning

Read-only software architect agent. Explores the codebase, designs a step-by-step implementation plan, identifies critical files. Use before `/batch` or any complex feature work.

---

### `/stuck` — Diagnose Stuck Sessions

Investigates other Claude Code processes on the machine for signs of freezing:
- High sustained CPU (infinite loop)
- Process state D/T/Z (I/O hang, stopped, zombie)
- Very high RSS (memory leak)
- Hung child processes (git, node, shell)

Reports findings with process details and debug log tail. Diagnostic only — never kills processes.

---

### `/remember` — Memory Review

Reviews your auto-memory entries and proposes promotions to the right layer:
- **CLAUDE.md** — project conventions all contributors should follow
- **CLAUDE.local.md** — personal preferences specific to you
- **Stay in auto-memory** — temporary or uncertain entries

Also detects duplicates, outdated entries, and conflicts across layers. Presents all proposals before making any changes.

---

## Design Philosophy

These skills share a few common principles:

**Adversarial posture over confirmation** — `/verify` assumes the implementation is wrong until proven otherwise. `/security-review` assumes vulnerabilities exist until the evidence says otherwise.

**Parallelism as a first-class concern** — `/simplify` and `/explore` fan out to multiple agents simultaneously. `/batch` isolates work into independent worktrees. Sequential execution is the exception, not the default.

**Proposals before changes** — `/remember` and `/plan` present findings for approval before touching anything. Read-only exploration is always safe; writes require explicit confirmation.

**Precision over completeness** — `/security-review` filters to >80% confidence findings. `/explore` omits sections that aren't relevant. Noise reduction is part of the design.

## License

MIT
