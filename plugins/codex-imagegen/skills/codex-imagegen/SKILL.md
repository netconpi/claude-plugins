---
name: codex-imagegen
description: Generate or edit raster images (photos, illustrations, product shots, UI mockups, textures, sprites, transparent cutouts, background removal, style transfer) through the Codex CLI's built-in image tool (gpt-image-2). Use when the user asks to generate, draw, render or edit a picture, or says "сгенерируй картинку", "нарисуй через кодекс", "сделай изображение". Claude calls it directly — no subagent, no delegation — then looks at the result and iterates.
---

# Generating images through Codex

Codex CLI has a built-in image tool (gpt-image-2). `codex-image` runs Codex only as an image backend —
read-only sandbox, no code changes — and copies the result to the path you choose. You write the spec,
look at the image, and iterate. Nothing is delegated: you are the art director and the judge.

Script: `"${CLAUDE_PLUGIN_ROOT}/scripts/codex-image"` (not on PATH; always use the full quoted path).

## When to use

- New bitmap: photo, illustration, concept art, product/packaging shot, website hero, texture, sprite, mockup.
- Edit of an existing image: background removal or replacement, object removal, relighting, style transfer,
  text localization, compositing.
- Not for: SVG/vector icons and logos that should match an existing set, diagrams, charts, wireframes
  or anything better built as code (SVG, HTML/CSS, canvas). Draw those yourself.
- Same judgment as drawing it yourself: no known characters, brand logos or real people's likenesses.

## How

1. Write the spec to a file. Use only the lines that help:

   ```text
   Use case: <photorealistic-natural | product-mockup | ui-mockup | illustration-story | stylized-concept | ...>
   Asset type: <where it will be used, e.g. landing page hero 16:9>
   Primary request: <the user's request>
   Subject: <main subject>
   Scene/backdrop: <environment>
   Style/medium: <photo / illustration / 3D render / watercolor ...>
   Composition/framing: <wide / close-up / top-down; aspect ratio; negative space>
   Lighting/mood: <lighting and mood>
   Color palette: <palette>
   Text (verbatim): "<exact in-image text>"
   Constraints: <must keep / must avoid; for edits: change only X, keep Y unchanged>
   Avoid: <negative constraints, e.g. no watermark, no extra text>
   ```

   If the user's prompt is detailed, normalize it; if it is vague, add only details that clearly help.

2. Run it with the Bash tool `timeout` set to 600000 ms (one image usually takes 1–3 minutes):

   ```bash
   "${CLAUDE_PLUGIN_ROOT}/scripts/codex-image" --out <dir>/<name>.png [--n 1-4] [--transparent] \
     [--ref style.png]... [--edit target.png]... < spec.md
   ```

   - `--edit` for the image being changed, `--ref` for style/composition references (5 in total at most).
   - `--n` makes variants of one spec. Distinct assets need distinct specs: run one command per asset,
     in parallel with `run_in_background` when there are several.
   - Without `--force` an existing file is never overwritten; a `-v2` sibling is written instead.

3. Read the output it prints (`status`, `images:` with paths and pixel size, `revised prompt`), then
   **look at every image with the Read tool** and check subject, composition, in-image text, constraints.

4. Iterate with one targeted change at a time: `--edit <previous.png>` plus a spec that repeats the
   invariants ("change only the sky; keep the house, framing and colors unchanged").

5. Tell the user the final path(s), show the image, and mention anything you could not get right.

## Sizes

The built-in tool picks the size itself (`auto`, roughly 1024–1536 px). Ask for the aspect ratio in the spec.
If an exact size is required, resize or crop afterwards (`sips -z <h> <w> file.png` on macOS, or Pillow).

## Failures

- `error: codex is not logged in` or no image with an "unavailable" error: the tool needs `codex login`
  with a paid ChatGPT plan (not an API key, not Free). Tell the user; do not fall back to drawing a
  bitmap substitute without saying so.
- Rate-limit / usage-limit errors: report them; do not retry in a loop.
- `status: partial`: some variants were saved; use them or rerun for the missing ones.
- Run logs: `~/.cache/codex-image/<run>/` (prompt sent to Codex, JSONL events, Codex's reply).
  Originals stay in `~/.codex/generated_images/<thread>/`.

## Configuration

`CODEX_IMAGE_MODEL` (default `gpt-6.1-sol`, the model that drives the tool call),
`CODEX_IMAGE_EFFORT` (default `low`), `CODEX_IMAGE_HOME`, `CODEX_HOME`.
