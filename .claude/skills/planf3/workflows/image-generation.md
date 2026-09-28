# Image Generation

Fill or update the embedded images in an existing plan `.html` file. Pick the sub-workflow based on the incoming `USER_PROMPT`:

| Sub-workflow | When to call it |
| --- | --- |
| Create | The prompt asks to generate, fill, or add the plan's images from scratch (empty `{{...IMAGE` slots) |
| Update | The prompt asks to change, refine, regenerate, or replace images that already exist in the plan |

Images come from the `codex-image` skill (`.claude/skills/codex-image/SKILL.md`): Codex's built-in image tool on the ChatGPT login, run in its own herdr pane. No API key. Check its prerequisites first (inside herdr, `codex login status` says ChatGPT); if they fail, stop and tell the user. Run from the project root:
- Create image: `uv run .claude/skills/codex-image/scripts/codex_image.py generate "<prompt>" <output.png>`
- Edit image: `uv run .claude/skills/codex-image/scripts/codex_image.py --replace edit "<instruction>" <image.png> <image.png>`

Shared rules for every image prompt:
- always ask for wide 16:9, high detail in the prompt text (there are no size or quality flags)
- convey the one or two core ideas of that section for a professional software engineer
- match the plan's synced visual identity (professional, focused, minimal)
- keep total words shown in the image under 10
- save images to `IMAGES_OUTPUT_DIR` (create it if missing)

## Create

1. Find slots - Grep the plan for `{{...IMAGE` placeholders (hero + per-phase). Each comment names the intended subject.
2. Write prompts - For each slot, write a prompt following the shared rules above.
3. Generate - Run `codex_image.py generate` once per slot, writing to `IMAGES_OUTPUT_DIR`.
4. Embed - Replace each `<!-- {{...IMAGE: ...}} -->` placeholder with `<img src="<plan-name>/<file>.png" alt="...">`, keeping the existing `<figure>`/`<figcaption>`.
5. Report - List the images generated and the slots filled.

## Update

1. Identify targets - From the `USER_PROMPT`, determine which embedded `<img>` images to change.
2. Write instruction - Write an edit instruction describing the change, following the shared rules above.
3. Edit - Run `codex_image.py --replace edit` with the existing PNG as both input and output; it replaces the file only if the new image is produced.
4. Verify embed - Confirm the `<img>` still points at the updated file; update `src`/`alt`/`<figcaption>` if the change warrants it.
5. Report - List the images updated and what changed.
