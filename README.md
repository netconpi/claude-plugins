# ntcad Claude Code plugins

## codex-delegation

The orchestrator delegates subagent work to the Codex CLI. A Claude `codex-worker` subagent (Sonnet) writes the brief, runs Codex, checks the result against the acceptance criteria, and resumes the same Codex thread with concrete gaps (up to 3 follow-up rounds). It returns `DONE` / `PARTIAL` / `FAILED` with evidence.

- `agents/codex-worker.md`: the judge (subagent type `codex-delegation:codex-worker`)
- `skills/codex-delegation/SKILL.md`: when and how the orchestrator delegates
- `scripts/codex-task`: background wrapper around `codex exec` (`start` / `wait` / `resume` / `status` / `log` / `stop`); runs live in `~/.cache/codex-task/`

Requirements on each machine: `codex` on PATH and logged in (`codex login status`). Works with macOS stock bash 3.2 and without setsid.
Model and effort: `CODEX_TASK_MODEL` (default `gpt-6.1-sol`), `CODEX_TASK_EFFORT` (default `high`).

### Install

```sh
claude plugin marketplace add netconpi/claude-plugins
claude plugin install codex-delegation@ntcad-plugins
```

Update after pulling changes: `claude plugin marketplace update ntcad-plugins && claude plugin update codex-delegation@ntcad-plugins`. Restart Claude Code sessions to pick up new agents.

Claude.ai and Cowork can't run this: their sandbox has no local `codex` CLI.

## kimi-delegation

Same shape as `codex-delegation`, with the [Kimi Code CLI](https://github.com/MoonshotAI/kimi-code) as the worker. A Claude `kimi-worker` subagent (Sonnet) writes the brief, runs Kimi, checks the result against the acceptance criteria, and resumes the same Kimi session with concrete gaps (up to 3 follow-up rounds). It returns `DONE` / `PARTIAL` / `FAILED` with evidence.

Modes (the orchestrator names one in the task):

- `default`: Kimi does the work itself (`kimi -p`, every tool call auto-approved, no OS sandbox).
- `read-only`: investigations and reviews; Kimi's Write/Edit tools are removed and shell writes are forbidden in the prompt (a guard, not a sandbox).
- `codex`: the ultimate mode, Claude → Kimi → Codex. Kimi plans, hands the heavy parts to Codex runs (in parallel git worktrees when useful), judges each run itself, merges and reports. `kimi-worker` then judges Kimi's result. `status` lists the Codex runs Kimi started; `stop` stops them too.

Files:

- `agents/kimi-worker.md`: the judge (subagent type `kimi-delegation:kimi-worker`)
- `skills/kimi-delegation/SKILL.md`: when and how the orchestrator delegates; triggers on "use Kimi", "через Kimi", "Kimi + Codex", "ульта"
- `scripts/kimi-task`: background wrapper around `kimi -p --output-format stream-json` (`start --mode default|read-only|codex` / `wait` / `resume` / `status` / `log` / `stop`); runs live in `~/.cache/kimi-task/`
- `scripts/agents/`: Kimi main-agent files for `read-only` and `codex` mode (passed with `--agent-file`)
- `kimi/codex-delegation/`: a Kimi-native plugin (skill + `codex-worker` judge + a copy of `codex-task`). Codex mode uses it without installation. To delegate to Codex from Kimi directly, install it in Kimi: `/plugins install <path to plugins/kimi-delegation/kimi/codex-delegation>`, then `/reload`.

Requirements on each machine: Kimi Code CLI 2.3+ logged in (`kimi login`), `jq` or `python3`; for `codex` mode also `codex` logged in. Works with macOS stock bash 3.2 and without setsid.
Env: `KIMI_TASK_MODEL` (default: `default_model` from `~/.kimi-code/config.toml`), `KIMI_TASK_HOME`, `CODEX_TASK_MODEL`, `CODEX_TASK_EFFORT`.

### Install

```sh
claude plugin marketplace update ntcad-plugins
claude plugin install kimi-delegation@ntcad-plugins
```

Claude.ai and Cowork can't run this either: their sandbox has no local `kimi` CLI.

## codex-imagegen

Lets Claude generate and edit images through the Codex CLI's built-in image tool (`image_gen`, gpt-image-2). No delegation and no judge subagent: Claude writes the spec, runs `codex-image`, looks at the result with Read and iterates (`--edit <previous.png>` with one targeted change). Codex runs as an image backend only, in a read-only sandbox, and the script copies the image to the path Claude chooses.

- `skills/codex-imagegen/SKILL.md`: when to generate (bitmaps, photo edits, cutouts) and when to draw as code instead; the spec template; the look-and-iterate loop
- `scripts/codex-image`: `codex-image --out FILE.png [--n 1-4] [--ref IMG]... [--edit IMG]... [--transparent] [--timeout SEC] [--force] < spec.md`. Synchronous (default timeout 540 s), never overwrites without `--force`, prints `status`, image paths with pixel size and the revised prompt; exit 0 / 3 partial / 1 failed. Images come from `~/.codex/generated_images/<thread>/`; logs in `~/.cache/codex-image/`.

Requirements on each machine: `codex` on PATH, logged in with a paid ChatGPT plan (the built-in image tool is not offered to API-key or Free logins). Size is chosen by the tool; ask for the aspect ratio in the spec and resize afterwards if needed.
Env: `CODEX_IMAGE_MODEL` (default `gpt-6.1-sol`), `CODEX_IMAGE_EFFORT` (default `low`), `CODEX_IMAGE_HOME`.

### Install

```sh
claude plugin marketplace update ntcad-plugins
claude plugin install codex-imagegen@ntcad-plugins
```
