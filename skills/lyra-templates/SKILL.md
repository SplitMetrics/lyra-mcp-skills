---
name: lyra-templates
description: Start from one of Lyra's ready-made templates — workflows built by Lyra's UGC creative producers (Hook, Talking Head, UGC, …) — and adapt it to the user's app or product. Use when the user asks for "a template", "something like the knitted doll / storm news / POV videos", "a proven ad format", "a UGC video / hook / talking head for my app", or wants a creative fast without designing the workflow. Prefer it over building from scratch whenever a template fits.
---

# lyra-templates

Templates are workflows Lyra's creative producers built and tested: the node graph, models, prompts and
pacing of a format that works. Copying one keeps that structure. Your job is to swap in the user's
subject and assets without breaking it.

## Steps

1. **Find candidates.**
   - `list_template_tags` gives the categories. Then call `list_templates` with `tags` and/or `search`.
   - Use `favourites_only: true` when the user says "my templates" or "the ones I starred".
   - Show 2–4 matches with their title, a one-line description and the `video_preview_url`, which the
     user opens while signed in to Lyra. Let the user pick, or recommend one and say why.
   - Prefer templates without upload nodes (`upload_image`, `upload_video`, `upload_audio`, `upload_documents`
     actions in `get_template`). Upload nodes aren't supported through the MCP yet, see below.
2. **Read it before copying.** Call `get_template`, then tell the user in plain words what it makes, for example
   "a 3-scene 9:16 video: hook image → animated clip → voiceover, ~15 s". Then sort its nodes into two groups:
   - **Subject slots:** prompts that name the example product, app, character or setting. These are what you rewrite.
   - **Craft:** camera, lighting, pacing, style, hook wording, transitions. Keep these.
   If `reference/<template>.md` exists for this template, read it. The producer's notes there override your
   judgement about what is safe to change.
3. **Gather what's missing.** Ask for the product or app name, what it does, the audience, the tone,
   and the CTA. Ask only for what the template's slots actually use.
4. **Copy.** Call `use_template` with a title like `<Template> – <Product>`. It returns the new workflow and its node uuids.
   Use those uuids from here on, not the template's.
5. **Adapt.** For each subject slot, call `update_node` with only the settings that change, usually `prompt`
   and sometimes `label`. Follow `reference/adapting-prompts.md`.
6. **Check.** Call `get_workflow` and fix every node whose `is_valid` is false. Show the user the rewritten prompts side by side
   with the originals. One look here costs less than a wasted video run.
7. **Confirm and run.** State the number of runs (generation nodes × `count`), the models, the estimated time and
   the workspace credits (`list_workspaces`), and wait for a yes. Then `run_node` in graph order, as the
   `lyra-start-here` rules describe: run sources first and finish them before running what they feed. Run independent nodes together.
   Reconnect image → video frames (`connect_nodes`, `role: first_frame`) after their image is generated.
8. **Deliver.** Give the final result `url`s and the workflow title, so the user can find it in Lyra.

## Gotchas

- **Upload nodes are not supported yet.** Don't call the `create_upload_*_node` tools. If the chosen template has upload
  nodes, they come out of `use_template` empty. Tell the user before copying: they'll need to add their files on those
  nodes in Lyra and run the nodes that depend on them there. Run only the nodes that don't depend on an upload.

- **Don't use `update_node` on a template's own nodes.** Template uuids only work with `get_template`. Edit the
  nodes of the workflow `use_template` returned.
- **If `use_template` reports that the copy was made but reading it failed,** don't call it again.
  That would make a second copy. Open the named workflow with `get_workflow`.
- **The copy has no brand kit.** Brand kits are not carried over. If the user has one, pick it with `list_brand_kits`
  and apply it with `assign_brand_kit` before running.
- **Talking-head and voiceover formats:** after copying, check in `get_workflow` that the video nodes still
  have their audio/voiceover inputs connected, because copying a template may drop them. Reconnect with `connect_nodes`
  (no `role`) before running.
- **Keep the model unless there's a reason to change it.** The producer picked it for this format. Switch it only when the user asks
  or the node is invalid, and then use only values from `list_node_configs`.
- **Text nodes with `action: user_text`** hold copy that is shown or spoken (hooks, captions, scripts). Rewrite them for the
  product. Keep their length and rhythm, because a hook that runs 2× longer breaks the timing.
- **Drafts:** templates the producer hasn't published never show up through the MCP, not even for admins. If the
  user names a template you can't find, it may still be a draft. They can see it in Lyra, but you can't use it until it's published.
