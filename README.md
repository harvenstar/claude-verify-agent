# claude-verify-agent

An adversarial verification skill for Claude Code, reverse-engineered from the internal `verificationAgent.ts` found in the Claude Code source leak.

## What This Is

Claude Code ships with a built-in Verification Agent that is used internally when running sub-agents. Its design philosophy is unusual: instead of confirming that code works, it **tries to break it**.

The source explicitly calls out two failure modes that most verification agents fall into:

1. **Verification Avoidance** — reading code instead of running checks, then declaring PASS
2. **Fooled by the First 80%** — the happy path works, tests pass, so you stop — missing the 20% of real failure cases

This repo extracts that adversarial verification behavior as a standalone Claude Code skill you can invoke with `/verify`.

## Installation

### Option A: Project-level (one project)

Copy the skill file into your project:

```bash
mkdir -p .claude/skills
curl -o .claude/skills/verify.md \
  https://raw.githubusercontent.com/YOUR_USERNAME/claude-verify-agent/main/.claude/skills/verify.md
```

### Option B: Global (all projects)

```bash
mkdir -p ~/.claude/skills
curl -o ~/.claude/skills/verify.md \
  https://raw.githubusercontent.com/YOUR_USERNAME/claude-verify-agent/main/.claude/skills/verify.md
```

Then restart Claude Code.

## Usage

After making changes, run:

```
/verify
```

Claude will:
1. Run build, tests, and linter — and show you the actual output
2. Run type-appropriate checks (frontend / backend / CLI / migrations)
3. Run at least 3 adversarial probes targeting edge cases the implementation might have missed
4. Output a final verdict:

```
VERDICT: PASS     — everything passed including adversarial probes
VERDICT: FAIL     — specific failures with exact commands and outputs
VERDICT: PARTIAL  — what passed, what didn't, what wasn't verified
```

## Why Adversarial?

Standard verification asks: *does this work?*
Adversarial verification asks: *how would this break?*

The difference matters because the implementer already has confirmation bias — they built the thing, they tested the happy path, they believe it works. A verification agent that shares that bias adds no value.

The internal Claude Code verificationAgent is explicitly designed to counteract implementer bias by assuming the implementation is wrong until proven otherwise.

## What's Different From Just Asking Claude to Test

When you ask Claude to "verify this works," it defaults to:
- Reading the code
- Running the obvious test
- Saying it looks fine

The `/verify` skill forces a different posture:
- Mandatory execution (no reading without running)
- Adversarial probes are not optional
- Every check must show exact command + exact output
- PARTIAL is not a soft PASS

## Source

Behavior reconstructed from `src/tools/AgentTool/verificationAgent.ts` in the Claude Code npm package sourcemap (`cli.js.map`), which contained full `sourcesContent` for 4,756 source files.

Original analysis: [claude-code-deep-dive](https://github.com/tvytlx/claude-code-deep-dive)

## License

MIT
