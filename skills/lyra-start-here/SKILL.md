---
name: lyra-start-here
description: Entry point for the Lyra (Muse) MCP server. Use when the user wants images, videos, voiceovers, music or text generated in Lyra, or a Lyra workflow inspected, built or run, and it is not yet clear which lyra-* skill fits. Routes to the right skill and states the rules every Lyra workflow follows. Does not generate on its own.
---

# Lyra — start here

The Lyra MCP server (`https://mcp-lyra.splitmetrics.com`) builds and runs **workflows**: graphs of
nodes (image, video, audio, text) joined by edges, in the workspace the user last selected
in Lyra. The tools give raw capability; these skills hold the order of calls, model choices and
known failures.

## Route by what the user wants

| User wants… | Load |
|---|---|
| A proven format adapted to their app: "a UGC video", "a hook", "like the knitted doll one", "a template" | `lyra-templates` ★ |
| One or a few images or videos from a prompt | `lyra-generate` |
| An image animated into a video (first/last frame, reference) | `lyra-image-to-video` |
| A person or character speaking a script (lip-sync) | `lyra-talking-head` |
| An existing image in other aspect ratios or languages | `lyra-adapt-image` |
| A multi-scene video ad: script → clips → voiceover/music → one composed video | `lyra-video-ad` |
| To change a prompt or setting on an existing node | `update_node` (keep the other settings) |
| To look at or rerun an existing workflow | `list_workflows` → `get_workflow`, then the skill that matches its nodes |

## Rules for every Lyra workflow

1. **Models come from `list_node_configs`, never from memory.** Use only a `model_name` and parameter
   values it lists for that node type, and only pass parameters that model has. Start from
   `preferred_models_by_action` when the user has no preference.
2. **Creating a node generates nothing; `run_node` spends credits.** Before the first run, say what
   will run (nodes × `count`, model, estimated time) and the workspace's credits from
   `list_workspaces`, then wait for a yes. One yes covers the plan you described, not later additions.
3. **`run_node` runs one node only.** Upstream nodes are not run for you. Run them first, in
   graph order, and wait for each to complete before running what depends on it.
4. **`get_run_results` already waits** (up to 45 s). Call it right after `run_node` with no pause. On
   `not_started` / `in_progress`, call it again straight away. Video runs take 1–6 minutes, so expect
   several calls. Never call `run_node` again because a run is slow; that starts and pays for a second run.
5. **Check `is_valid` / `validation_error`** on every node a create or connect tool returns, and fix
   them before `run_node`.
6. **Result links are signed by Lyra cookies.** Give the user the `url`s. They open in a browser where
   the user is signed in to Lyra. Do not fetch or embed them.
7. **Do not delete what the user did not ask to delete.** `delete_node` cannot be undone.

★ When the user wants an ad or social creative and hasn't designed a workflow, check `list_templates` first.
A template built and tested by Lyra's creative producers beats a workflow built from scratch.

## Workspace and workflow

- Workflows live in the workspace the user last selected in Lyra. The server cannot switch
  workspace, so if `list_workspaces` shows several, tell the user which workspace is active and that
  they switch it in Lyra.
- Reuse a workflow when the user names one (`list_workflows` with `search`). Otherwise
  `create_workflow` with a title that describes the job, so the user can find it in Lyra.

## The user's own files are not supported yet

Do not call the `create_upload_image_node`, `create_upload_video_node`, `create_upload_audio_node` or
`create_upload_documents_node` tools, and don't build workflows that depend on upload nodes. If the user wants to use
their own photo, video, audio or document, tell them this isn't supported through the MCP yet. Offer to
generate the asset instead, or let them do that part in Lyra themselves.

## What this skill does not do

It does not generate anything. Once routed, load that skill and follow it.
