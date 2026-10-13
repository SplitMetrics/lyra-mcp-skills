---
name: lyra-generate
description: Generate images or videos from a text prompt in Lyra. Use when the user asks Lyra for "an image of…", "a few variations", "a 9:16 video of…", or several prompts at once, with no source image to animate. Covers model choice, parameters, running several nodes at once, and collecting results. For animating an image use lyra-image-to-video; for speech use lyra-talking-head.
---

# lyra-generate

## Steps

1. **Workflow.** Reuse one the user named (`list_workflows` with `search`), or `create_workflow`.
2. **Model.** `list_node_configs` → node type `image` or `video`. Take the user's model if they named one.
   Otherwise pick by the table below and check the model is still listed.
3. **Nodes.** One `create_image_generation_node` / `create_video_generation_node` per distinct prompt.
   Use `count` for variations of the same prompt, not extra nodes. Give each node a `label` the user
   will recognise on the canvas.
4. **Confirm.** State the number of nodes, `count`, model, estimated time (from the model's
   `estimated_generation_time`) and current workspace credits (`list_workspaces`). Then wait for a yes.
5. **Run everything, then collect.** Call `run_node` for **every** node first. Keep each returned
   `run_uuid` and `variation_uuid`. Only then call `get_run_results` for each. Lyra runs them in
   parallel. If you run and wait on one node at a time, a batch takes as long as the sum of all its runs.
6. **Report.** List each node's label and its result `url`s. Name any failed node and say what it was
   asked to do.

## Choosing a model (defaults when the user has no preference)

| Need | Image | Video |
|---|---|---|
| General, fast | `google/nano-banana-2` | `kwaivgi/kling-v3-video-standard` |
| Highest quality | `google/nano-banana-pro`, `black-forest-labs/flux-2-max` | `kwaivgi/kling-v3-video`, `google/veo-3.1` |
| Legible text / typography in the image | `ideogram-ai/ideogram-v3-quality` | — |
| Transparent background | `openai/gpt-image-2` with `background: transparent` | — |
| Cheap/fast drafts | `google/nano-banana`, `ideogram-ai/ideogram-v3-turbo` | `google/veo-3.1-lite`, `bytedance/seedance-2.0-mini` |
| Native audio in the clip | — | Kling 3.0, Veo 3.1, Seedance 2.x (`generate_audio`) |

This table is a starting point. `list_node_configs` is the source of truth, and its
`preferred_models_by_action` order reflects what Lyra recommends now.

## Parameters

- **Aspect ratio:** most models default to `9:16` (mobile ads). Pass `aspect_ratio` when the user wants
  anything else, and only use values in that model's enum. Some models take ratios through
  `resolution` instead (Wan: `1280*720`), and some cap the choice (GPT Image 2: `1:1`, `3:2`, `2:3`).
- **Video duration:** pass a value from the model's `duration` enum or range. If you omit it, the
  model's own default is used. `adaptive` (and Seedance's `-1`) let the model choose.
  Long durations multiply cost.
- **Don't send what the model lacks:** e.g. Sora 2 has no `generate_audio` or `resolution`, Veo
  3.1 Lite has no `generate_audio`.

## Prompts

Write the prompt as a shot description: subject, action, setting, lighting, camera/lens, style.
For video, add motion and camera movement ("slow push-in", "handheld"). When the user gives a short
idea, expand it, show them the expanded prompt in your confirmation, and keep their wording for the subject.

## Gotchas

- `validation_error` on a new node usually means an option value the model doesn't accept. Recreate
  the node with a listed value. A node created with a bad option is not repaired by running it.
- **A re-run with an unchanged prompt is a re-roll, not a fix.** If the result was wrong, change the prompt or the model and say what you changed.
- Do not call `run_node` again while `get_run_results` reports `in_progress`. That starts a second paid run.
