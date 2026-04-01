---
name: skillify
description: Captures the current session's repeatable process as a reusable Claude Code skill file — interviews you to confirm name, steps, and success criteria, then writes the skill to .claude/skills/
allowed-tools: Bash, Read, Write, Glob, Grep
user-invocable: true
---

# Skillify

You are capturing this session's repeatable process as a reusable skill.

## Step 1: Analyze the Session

Before asking any questions, analyze the conversation history to identify:
- What repeatable process was performed
- What the inputs/parameters were
- The distinct steps (in order)
- The success artifacts/criteria for each step (e.g. not just "writing code," but "an open PR with CI fully passing")
- Where the user corrected or steered you
- What tools and permissions were needed
- What agents were used

## Step 2: Interview the User

Use AskUserQuestion for ALL questions. Never ask questions via plain text.

**Round 1: High-level confirmation**
- Suggest a name and description based on your analysis
- Ask the user to confirm or rename

**Round 2: Steps and inputs**
- Present the ordered steps you identified
- Ask if any are missing, wrong, or should be reordered
- Ask what parameters/inputs the skill should accept (if any)

**Round 3: Success criteria**
- Confirm the definition of "done" for this skill
- Ask if there are any common failure modes to document

## Step 3: Write the Skill

Once confirmed, write the skill to `.claude/skills/<name>.md` with this format:

```markdown
---
name: <name>
description: <one-line description>
allowed-tools: <comma-separated list of tools needed>
user-invocable: true
---

# <Name>

<brief description of what this skill does>

## Inputs

<list any parameters the user can provide>

## Steps

<numbered list of steps, written as instructions to Claude>

## Success Criteria

<what done looks like>
```

Then tell the user:
- Where the skill was written
- How to invoke it: `/<name>` or by asking Claude to use it
- That they can edit the file to refine it further
