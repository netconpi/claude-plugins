---
name: codex-delegation
description: Delegate implementation, investigation, refactoring, test or review work to the Codex CLI (gpt-6.1-sol) through a codex-worker subagent that runs Codex, judges the result and pushes Codex until the acceptance criteria pass.
whenToUse: When the user says "use Codex", "через кодекс", "delegate to Codex", or when a task is big enough that you would otherwise hand it to a coder subagent.
---

# Delegating work to Codex

You are the orchestrator. Work you would normally hand to a subagent is executed by the Codex CLI;
a `codex-worker` subagent sits in between as the judge.

```
you (orchestrator) ──Agent(subagent_type: "codex-worker")──► judge ──codex-task──► Codex CLI
                   ◄──────── verdict + evidence ─────────────┘  ◄── result / follow-up rounds
```

The wrapper script lives at `${KIMI_SKILL_DIR}/scripts/codex-task` (start / wait / resume / status / log / stop).

## When to use

- Any task you would give a subagent: implementation slices, bug fixes, test writing, refactors,
  codebase investigations, code reviews.
- Keep doing yourself: quick reads, one-line edits, decisions that need the user, anything a few tool calls settle.

## How to delegate

Call the `Agent` tool with `subagent_type: "codex-worker"`. The prompt is the task, and it must stand alone
(the judge and Codex do not see this conversation). Start it with the wrapper path, then:

0. `codex-task: ${KIMI_SKILL_DIR}/scripts/codex-task` — always the first line, exactly like this.
1. **Goal** and why it matters.
2. **Where**: absolute repo path (or worktree path), files/areas involved.
3. **Acceptance criteria**: numbered and checkable (commands that must pass, behavior to observe).
4. **Boundaries**: files not to touch; no commit/push/deploy unless intended.
5. **Sandbox** if not the default: `read-only` for investigations/reviews, `danger-full-access`
   only when network or outside-repo writes are needed.

## Parallel work

- Run several `codex-worker` subagents with `run_in_background: true`, then collect them with `WaitFor`.
- Read-only tasks can run in parallel in the same checkout.
- Writing tasks in parallel must each get their own git worktree, otherwise Codex runs overwrite each other.
  Create them before spawning: `git -C <repo> worktree add ../<repo>-<slug> -b codex/<slug>`
  and pass that path as the working dir. Merge the branches yourself after verifying the verdicts.

## After the verdict

- `DONE` — still verify the key claim yourself (diff, one test run) before telling the user it is done.
- `PARTIAL` / `FAILED` — read the open issues; re-delegate with a sharper brief, fix it yourself, or ask the user.
- Report Codex's work to the user faithfully, including what the judge could not verify.

## If the `codex-worker` subagent is not available

Spawn a `coder` subagent instead and put this in its prompt: "Follow the protocol in
${KIMI_SKILL_DIR}/../../agents/codex-worker.md — you are the judge it describes; never edit project files yourself."
plus the task. If the `codex` binary is missing or not logged in (`codex login status`), tell the user;
do not silently do the work yourself or with other subagents.

## Configuration

- Model and effort for Codex: env `CODEX_TASK_MODEL` (default `gpt-6.1-sol`), `CODEX_TASK_EFFORT` (default `high`).
- Run logs: `~/.cache/codex-task/<run>/round-N/` (prompt, JSONL events, final message).
