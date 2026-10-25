---
name: codex-delegation
description: Delegate subagent work to the Codex CLI (gpt-6.1-sol) instead of Claude subagents. Use whenever you would spawn a subagent to implement, investigate, refactor, test or review something, or when the user says "use Codex", "через кодекс", "delegate to Codex". Each delegated task goes to a codex-worker subagent that runs Codex, judges the result and pushes Codex until the acceptance criteria pass.
---

# Delegating work to Codex

You are the orchestrator. Work you would normally hand to a subagent is executed by Codex CLI;
a Claude `codex-delegation:codex-worker` subagent sits in between as the judge.

```
you (orchestrator) ──Agent(subagent_type: "codex-delegation:codex-worker")──► judge ──codex-task──► Codex CLI
                   ◄──────── verdict + evidence ────────────┘  ◄── result / follow-up rounds
```

## When to use

- Any task you would give a subagent: implementation slices, bug fixes, test writing, refactors,
  codebase investigations, code reviews.
- Keep doing yourself: quick reads, one-line edits, decisions that need the user, anything a few tool calls settle.

## How to delegate

Spawn `Agent` with `subagent_type: "codex-delegation:codex-worker"`. The prompt is the task, and it must stand alone
(the judge and Codex do not see this conversation):

1. **Goal** and why it matters.
2. **Where**: absolute repo path (or worktree path), files/areas involved.
3. **Acceptance criteria**: numbered and checkable (commands that must pass, behavior to observe).
4. **Boundaries**: files not to touch; no commit/push/deploy unless intended.
5. **Sandbox** if not the default: `read-only` for investigations/reviews, `danger-full-access`
   only when network or outside-repo writes are needed.

## Parallel work

- Read-only tasks can run in parallel in the same checkout.
- Writing tasks in parallel must each get their own git worktree, otherwise Codex runs overwrite each other.
  Create them before spawning:
  `git -C <repo> worktree add ../<repo>-<slug> -b codex/<slug>` and pass that path as the working dir.
  Merge the branches yourself after verifying the verdicts.

## After the verdict

- `DONE` — still verify the key claim yourself (diff, one test run) before telling the user it is done.
- `PARTIAL` / `FAILED` — read the open issues; re-delegate with a sharper brief, fix it yourself, or ask the user.
- Report Codex's work to the user faithfully, including what the judge could not verify.

## If `codex-delegation:codex-worker` is not available

In an environment without the subagent definition, spawn a `general-purpose` agent and include in its
prompt: "Follow the protocol in ${CLAUDE_PLUGIN_ROOT}/agents/codex-worker.md (the codex-task script is ${CLAUDE_PLUGIN_ROOT}/scripts/codex-task)" plus the task. If the `codex` binary is
missing or not logged in (`codex login status`), tell the user; do not silently fall back to Claude subagents.

## Configuration

- Wrapper: `${CLAUDE_PLUGIN_ROOT}/scripts/codex-task` (start / wait / resume / status / log / stop).
- Model and effort: env `CODEX_TASK_MODEL` (default `gpt-6.1-sol`), `CODEX_TASK_EFFORT` (default `high`).
- Run logs: `~/.cache/codex-task/<run>/round-N/` (prompt, JSONL events, final message).
