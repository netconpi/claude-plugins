# ntcad Claude Code plugins

## codex-delegation

The orchestrator delegates subagent work to the Codex CLI. A Claude `codex-worker` subagent (Sonnet) writes the brief, runs Codex, checks the result against the acceptance criteria, and resumes the same Codex thread with concrete gaps (up to 3 follow-up rounds). It returns `DONE` / `PARTIAL` / `FAILED` with evidence.

- `agents/codex-worker.md`: the judge (subagent type `codex-delegation:codex-worker`)
- `skills/codex-delegation/SKILL.md`: when and how the orchestrator delegates
- `scripts/codex-task`: background wrapper around `codex exec` (`start` / `wait` / `resume` / `status` / `log` / `stop`); runs live in `~/.cache/codex-task/`

Requirements on each machine: `codex` on PATH and logged in (`codex login status`), plus bash and setsid.
Model and effort: `CODEX_TASK_MODEL` (default `gpt-6.1-sol`), `CODEX_TASK_EFFORT` (default `high`).

### Install

```sh
claude plugin marketplace add <git-url-or-local-path-of-this-repo>
claude plugin install codex-delegation@ntcad-plugins
```

Update after pulling changes: `claude plugin marketplace update ntcad-plugins && claude plugin update codex-delegation@ntcad-plugins`. Restart Claude Code sessions to pick up new agents.

Claude.ai and Cowork can't run this: their sandbox has no local `codex` CLI.
