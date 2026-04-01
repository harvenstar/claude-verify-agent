# claude-verify-agent

A collection of Claude Code skills reverse-engineered from the internal source of Claude Code. Each skill is a markdown file you can drop into any project — no installation required beyond copying the file.

## Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| [verify](.claude/skills/verify.md) | `/verify` | Adversarial verification agent — tries to break your implementation, not confirm it |
| [simplify](.claude/skills/simplify.md) | `/simplify` | 3 parallel code review agents: reuse, quality, efficiency |
| [batch](.claude/skills/batch.md) | `/batch` | Orchestrate large parallelizable changes across a codebase in isolated worktrees |
| [security-review](.claude/skills/security-review.md) | `/security-review` | 3-phase security review with false-positive filtering (>80% confidence threshold) |
| [pr-comments](.claude/skills/pr-comments.md) | `/pr-comments` | Fetch and display all comments from the current branch's GitHub PR |
| [skillify](.claude/skills/skillify.md) | `/skillify` | Capture this session's repeatable process as a reusable skill file |
| [explore](.claude/skills/explore.md) | `/explore` | Deep read-only codebase exploration — parallel, structured report |
| [plan](.claude/skills/plan.md) | `/plan` | Software architect agent — design an implementation plan before writing code |

## Installation

### Global (available in all projects)

```bash
mkdir -p ~/.claude/skills
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/verify.md -o ~/.claude/skills/verify.md
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/simplify.md -o ~/.claude/skills/simplify.md
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/batch.md -o ~/.claude/skills/batch.md
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/security-review.md -o ~/.claude/skills/security-review.md
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/pr-comments.md -o ~/.claude/skills/pr-comments.md
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/skillify.md -o ~/.claude/skills/skillify.md
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/explore.md -o ~/.claude/skills/explore.md
curl -s https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/plan.md -o ~/.claude/skills/plan.md
```

### One-liner (all skills at once)

```bash
mkdir -p ~/.claude/skills && for skill in verify simplify batch security-review pr-comments skillify explore plan; do curl -s "https://raw.githubusercontent.com/harvenstar/claude-verify-agent/main/.claude/skills/$skill.md" -o ~/.claude/skills/$skill.md; done
```

### Project-level (one project only)

```bash
mkdir -p .claude/skills
# then curl individual skills as above, replacing ~/.claude with .claude
```

Restart Claude Code after installing.

## Skill Details

### `/verify` — Adversarial Verification

Most verification agents confirm the happy path. This one tries to break your implementation.

Reconstructed from `src/tools/AgentTool/built-in/verificationAgent.ts`. The internal design calls out two failure modes it explicitly guards against:
- **Verification Avoidance**: reading code and declaring PASS without running anything
- **Fooled by First 80%**: happy path passes, so you stop

Every check must show exact command + exact output. Output is always one of:
```
VERDICT: PASS | FAIL | PARTIAL
```

---

### `/simplify` — Code Review (3 Parallel Agents)

Reconstructed from `src/skills/bundled/simplify.ts`. Launches three agents in parallel:
1. **Code Reuse** — finds existing utilities that replace newly written code
2. **Code Quality** — catches redundant state, copy-paste, stringly-typed patterns, unnecessary comments
3. **Efficiency** — catches N+1 queries, missed concurrency, hot-path bloat, memory leaks

---

### `/batch` — Parallel Work Orchestration

Reconstructed from `src/skills/bundled/batch.ts`. For large refactors:
1. Decomposes work into 5–30 independent units
2. Spawns worker agents in isolated git worktrees
3. Each worker implements, tests, commits, and opens a PR
4. Coordinator aggregates all PR URLs

---

### `/security-review` — Security Review

Reconstructed from `src/commands/security-review.ts` (an internal Anthropic-only command moved to paid plugin). 3-phase methodology:
1. Repository context research
2. Comparative analysis against existing patterns
3. Vulnerability assessment

Only reports HIGH/MEDIUM findings with >80% confidence. 14 hard exclusions (DoS, rate limiting, outdated deps, etc.) to minimize false positives.

---

### `/pr-comments` — PR Comments

Reconstructed from `src/commands/pr_comments/index.ts` (also moved to paid plugin). Fetches both PR-level and inline code review comments from the current branch's GitHub PR via the `gh` CLI. Displays them with diff context.

---

### `/skillify` — Session to Skill

Reconstructed from `src/skills/bundled/skillify.ts`. Interviews you about the current session's repeatable process, then writes a `.claude/skills/<name>.md` file you can invoke later.

---

### `/explore` — Codebase Exploration

Reconstructed from `src/tools/AgentTool/built-in/exploreAgent.ts`. Read-only, fully parallelized exploration. Returns a structured report: architecture, relevant files, key patterns, gotchas, suggested entry points.

---

### `/plan` — Implementation Planning

Reconstructed from `src/tools/AgentTool/built-in/planAgent.ts`. Read-only software architect agent. Explores the codebase, designs an implementation plan, identifies critical files. Use this before `/batch` or complex features.

---

## Source

All skills reconstructed from `src/` in the Claude Code npm package sourcemap (`cli.js.map`), which contained full `sourcesContent` for 4,756 TypeScript source files.

Original analysis: [claude-code-deep-dive](https://github.com/tvytlx/claude-code-deep-dive)

## License

MIT
