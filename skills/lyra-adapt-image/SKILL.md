---
name: lyra-adapt-image
description: Adapt an existing Lyra image to other aspect ratios or translate the text on it into other languages, keeping the design. Use when the user wants "this banner in 1:1, 4:5 and 16:9", "resize for stories/feed", "localize this ad into German and Japanese", or "translate the text on the creative".
---

# lyra-adapt-image

Both adaptations take a **connected source image** and produce one output per target: one per aspect ratio, or one per language.

## Steps

1. **Source.** An image node already in the workflow (`get_workflow`), a newly generated one (run it first), or the user's file through `create_upload_image_node`. If the node has several results, ask which one to use.
2. **Model.** `list_node_configs` → `image.preferred_models_by_action.image_resize` / `image_translation` (currently `google/nano-banana-2`).
3. **Node.**
   - Resize: `create_image_resize_node` with `aspect_ratios`, e.g. `["1:1", "4:5", "16:9"]`.
   - Translate: `create_image_translation_node` with `languages`, using only codes from `image.action_parameters.image_translation.languages` (e.g. `de`, `ja`, `pt`, `zh-CN`, `zh-TW`).
4. **Connect** the source → the new node with `connect_nodes`, without `role`.
5. **Confirm** the number of outputs (ratios or languages) and wait for a yes. Then `run_node` → `get_run_results`.

## Combining both

To get every language in every size, translate first, then resize each translated result. Resizing first and then translating every size works too, but uses more runs. Count the outputs (languages × ratios) and state the total before running.

## Gotchas

- Resize re-renders the image generatively. It extends or recomposes the scene and does not crop. For pixel-exact approved assets (legal-approved pack shots, logos), say so and suggest a plain crop outside Lyra.
- Translation rewrites only the text it finds. Check that brand names and prices were not translated and that the right-to-left languages (`ar`, `he`, `fa`, `ur`) are laid out correctly, and point out problems to the user.
- Language codes are case-sensitive (`zh-CN`, not `zh-cn`).
