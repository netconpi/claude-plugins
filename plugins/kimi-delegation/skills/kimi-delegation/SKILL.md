---
name: kimi-delegation
description: Delegate subagent work to the Kimi Code CLI instead of Claude subagents, and in ultimate (codex) mode let Kimi orchestrate and judge Codex CLI runs — Claude → Kimi → Codex. Use when the user says "use Kimi", "через Kimi", "через кими", "delegate to Kimi", "Kimi + Codex", "ульта", "ultimate mode", or asks for Kimi to drive Codex. Each delegated task goes to a kimi-worker subagent that runs Kimi, judges the result and pushes Kimi until the acceptance criteria pass.
---

# Delegating work to Kimi (and through Kimi to Codex)

You are the orchestrator. Work you would normally hand to a subagent is executed by the Kimi Code CLI;
a Claude `kimi-delegation:kimi-worker` subagent sits in between as the judge.

```
you ──Agent(subagent_type: "kimi-delegation:kimi-worker")──► judge ──kimi-task──► Kimi Code CLI
    ◄──────────── verdict + evidence ─────────────┘  ◄── result / follow-up rounds

ultimate (--mode codex):
you ──► kimi-worker (judge) ──kimi-task──► Kimi (orchestrator + judge) ──codex-task──► Codex CLI ×N
```

## Modes

| mode | who does the work | use for |
| --- | --- | --- |
| `default` | Kimi itself | implementation, fixes, tests, refactors |
| `read-only` | Kimi, with Write/Edit removed | investigations, code reviews, "explain this" |
| `codex` | Kimi plans and judges, Codex executes | big or parallelizable tasks; when the user wants Kimi + Codex |

In `codex` mode Kimi can also generate raster images through Codex's built-in image tool
(the bundled `codex-image` script); say so in the brief when the task needs images.

## When to use

- The user asks for Kimi, or for the Kimi + Codex ultimate mode.
- Any task you would give a subagent, once the user has chosen Kimi for this session.
- Keep doing yourself: quick reads, one-line edits, decisions that need the user, anything a few tool calls settle.

## How to delegate

Spawn `Agent` with `subagent_type: "kimi-delegation:kimi-worker"`. The prompt is the task, and it must stand alone
(the judge and Kimi do not see this conversation):

1. **Mode**: `default`, `read-only` or `codex`.
2. **Goal** and why it matters.
3. **Where**: absolute repo path (or worktree path), files/areas involved.
4. **Acceptance criteria**: numbered and checkable (commands that must pass, behavior to observe).
5. **Boundaries**: files not to touch; no commit/push/deploy unless intended. Kimi has no OS sandbox —
   in `default` and `codex` mode it can write anywhere your user can, so be explicit.

## Parallel work

- Read-only tasks can run in parallel in the same checkout.
- Writing tasks in parallel must each get their own git worktree, otherwise runs overwrite each other.
  Create them before spawning:
  `git -C <repo> worktree add ../<repo>-<slug> -b kimi/<slug>` and pass that path as the working dir.
  Merge the branches yourself after verifying the verdicts.
- In `codex` mode one Kimi run already fans out into parallel Codex runs; prefer one codex-mode task per
  coherent goal over many small ones.

## After the verdict

- `DONE` — still verify the key claim yourself (diff, one test run) before telling the user it is done.
- `PARTIAL` / `FAILED` — read the open issues; re-delegate with a sharper brief, fix it yourself, or ask the user.
- Report the work to the user faithfully — who did what (Kimi, Codex runs), and what the judge could not verify.

## If `kimi-delegation:kimi-worker` is not available

In an environment without the subagent definition, spawn a `general-purpose` agent and include in its
prompt: "Follow the protocol in ${CLAUDE_PLUGIN_ROOT}/agents/kimi-worker.md (the kimi-task script is
${CLAUDE_PLUGIN_ROOT}/scripts/kimi-task)" plus the task. If the `kimi` binary is missing or not logged in,
tell the user (`kimi login`); do not silently fall back to Claude subagents.

## Configuration

- Wrapper: `${CLAUDE_PLUGIN_ROOT}/scripts/kimi-task` (start / wait / resume / status / log / stop).
- Kimi model: env `KIMI_TASK_MODEL` (default: `default_model` from `~/.kimi-code/config.toml`).
- Codex in ultimate mode: env `CODEX_TASK_MODEL` (default `gpt-6.1-sol`), `CODEX_TASK_EFFORT` (default `high`).
- Run logs: `~/.cache/kimi-task/<run>/round-N/` (prompt, stream-json events, final message);
  Codex runs started by Kimi: `~/.cache/codex-task/<run>/`.
- The Kimi-side Codex skill ships in `${CLAUDE_PLUGIN_ROOT}/kimi/codex-delegation`. Codex mode uses it
  without installation; to use it from Kimi directly, install it there with `/plugins install <that path>`.
