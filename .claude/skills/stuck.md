---
name: stuck
description: Diagnose frozen, stuck, or very slow Claude Code sessions on this machine — reports CPU, memory, process state, and child processes
allowed-tools: Bash
user-invocable: true
---

# /stuck — diagnose frozen/slow Claude Code sessions

The user thinks another Claude Code session on this machine is frozen, stuck, or very slow. Investigate and produce a diagnostic report.

## What to look for

Scan for other Claude Code processes (excluding the current one). Process names are typically `claude` (installed) or `cli` (native dev build).

Signs of a stuck session:
- **High CPU (≥90%) sustained** — likely an infinite loop. Sample twice, 1-2s apart, to confirm it's not a transient spike.
- **Process state `D` (uninterruptible sleep)** — often an I/O hang. The `state` column in `ps` output; first character matters (ignore modifiers like `+`, `s`, `<`).
- **Process state `T` (stopped)** — user probably hit Ctrl+Z by accident.
- **Process state `Z` (zombie)** — parent isn't reaping.
- **Very high RSS (≥4GB)** — possible memory leak making the session sluggish.
- **Stuck child process** — a hung `git`, `node`, or shell subprocess can freeze the parent. Check `pgrep -lP <pid>` for each session.

## Investigation steps

1. **List all Claude Code processes** (macOS/Linux):
   ```
   ps -axo pid=,pcpu=,rss=,etime=,state=,comm=,command= | grep -E '(claude|cli)' | grep -v grep
   ```
   Filter to rows where `comm` is `claude` or (`cli` AND the command path contains "claude").

2. **For anything suspicious**, gather more context:
   - Child processes: `pgrep -lP <pid>`
   - If high CPU: sample again after 1-2s to confirm it's sustained
   - If a child looks hung (e.g., a git command), note its full command line with `ps -p <child_pid> -o command=`
   - Check the session's debug log: `~/.claude/debug/<session-id>.txt` (last few hundred lines show what it was doing before hanging)

3. **Consider a stack dump** for a truly frozen process (advanced, optional):
   - macOS: `sample <pid> 3` gives a 3-second native stack sample
   - Only grab this if the process is clearly hung and you want to know *why*

## Report format

If everything looks healthy, tell the user directly — no further action needed.

If you found a stuck/slow session, output a structured report:

```
## Stuck Session Report

**PID**: <pid>
**CPU**: <cpu>%  **RSS**: <rss>MB  **State**: <state>  **Uptime**: <etime>
**Command**: <full command line>
**Child processes**: <list or "none">

**Diagnosis**: <what's likely wrong>

**Debug log tail** (if available):
<last 20 lines of ~/.claude/debug/<session-id>.txt>

**Stack sample** (if captured):
<sample output>
```

## Rules
- Do NOT kill or signal any processes — diagnostic only.
- If the user gave a specific PID or symptom as an argument, focus there first.
