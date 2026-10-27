---
name: codex-image
description: Generate or edit raster images (photos, illustrations, product shots, UI mockups, textures, sprites, transparent cutouts, background removal, style transfer) through the Codex CLI's built-in image tool (gpt-image-2). You call it directly and judge the result yourself; no subagent.
whenToUse: When the user asks to generate, draw, render or edit a picture ("сгенерируй картинку", "нарисуй", "make an image"), or a task needs a bitmap asset. Not for SVG icons, diagrams or charts that are better written as code.
---

# Generating images through Codex

The wrapper `${KIMI_SKILL_DIR}/scripts/codex-image` runs Codex only as an image backend (read-only
sandbox, no code changes) and copies the result to the path you choose. You write the spec, look at
the image, and iterate.

## When to use

- New bitmap: photo, illustration, concept art, product shot, website hero, texture, sprite, mockup.
- Edit of an existing image: background removal or replacement, object removal, relighting, style transfer.
- Not for SVG/vector assets that should match an existing set, diagrams, charts or wireframes: write those as code.
- No known characters, brand logos or real people's likenesses.

## How

1. Write a short labeled spec to a file (use only the lines that help): Use case, Asset type,
   Primary request, Subject, Scene/backdrop, Style/medium, Composition/framing (aspect ratio),
   Lighting/mood, Color palette, Text (verbatim), Constraints, Avoid. For edits, list what must stay unchanged.

2. Run it with `run_in_background: true` (one image takes 1–3 minutes, longer than a foreground Bash call allows):

   ```bash
   "${KIMI_SKILL_DIR}/scripts/codex-image" --out <dir>/<name>.png [--n 1-4] [--transparent] [--ref style.png] [--edit target.png] < spec.md
   ```

   The result arrives when the task finishes; use `WaitFor` if you need it within the current turn.
   `--edit` marks the image being changed, `--ref` a style/composition reference (5 in total at most).
   Distinct assets need distinct runs; `--n` only makes variants of one spec. An existing file is never
   overwritten without `--force`.

3. Read the printed report (`status`, image paths with pixel size, revised prompt), then open every image
   with `ReadMediaFile` and check subject, composition, in-image text and constraints.

4. Iterate one change at a time: `--edit <previous.png>` with a spec that repeats the invariants.

5. Report the final paths and anything you could not get right.

## Notes

- Size is chosen by the tool (auto); ask for the aspect ratio in the spec and resize afterwards if an exact size is needed.
- Needs `codex login` with a paid ChatGPT plan. If the tool is unavailable or rate-limited, say so; do not retry in a loop.
- Logs: `~/.cache/codex-image/<run>/`; originals stay in `~/.codex/generated_images/<thread>/`.
