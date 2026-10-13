---
name: lyra-image-to-video
description: Animate an image into a video in Lyra, using a generated image as the first frame, last frame, or reference. Use when the user asks to "animate", "bring to life", "make a video from this image", "start the video on this frame", or wants a transition between two images.
---

# lyra-image-to-video

## Steps

1. **Workflow.** Reuse one or `create_workflow`.
2. **Get the image node.**
   - Generate it: `create_image_generation_node` with the video's aspect ratio (see Gotchas), then `run_node` → `get_run_results` until `completed`.
   - Or an existing node: `get_workflow` → its uuid.
3. **Pick the result.** If the image node made several images, show them and ask which to use, *before* connecting. The user selects it in Lyra.
4. **Video node.** `list_node_configs` → a video model whose parameters include the input you need (table below). `create_video_generation_node` with a prompt that describes **motion only**: what moves and how the camera moves. The image already defines the look.
5. **Connect.** `connect_nodes` from the image to the video with `role`:
   - `first_frame`: the video starts on the image (most common).
   - `last_frame`: the video ends on it. Connect a second image as `first_frame` for an A → B transition.
   - `reference`: the images guide the subject or style but are not frames.
   Check the returned node's `is_valid`.
6. **Confirm and run.** Give the model, duration and estimated time (1–6 min), and wait for a yes. Then `run_node` on the video node → `get_run_results`, repeating while `in_progress`.

## Which video models take which input

| Input | Models (check `list_node_configs` for the current list) |
|---|---|
| `first_frame` | almost all: Kling, Seedance, Veo 3.1, Sora 2, Runway Gen 4.5 |
| `first_frame` + `last_frame` | Kling 3.0 / 3.0 Standard / Omni / 2.5 Turbo, Seedance 1.5–2.5, Veo 3.1 / Fast / Lite |
| `reference` images | Seedance 2.x (not with frames), Kling 3.0 Omni, Veo 3.1 (16:9, 8 s only), Gemini Omni video |

Pick a model that has the parameter. A role the model lacks is ignored or invalid.

## Gotchas

- **Frames need a finished image.** `connect_nodes` with `first_frame`/`last_frame` fails if the image node has no generated result yet. Run the image first, then connect.
- **Frames are pinned to the image that is selected when you connect.** If the user later picks a different image or reruns the image node, call `connect_nodes` again so the video uses the new one.
- **Match aspect ratios.** Sora 2 requires the frame to have the video's aspect ratio. Kling and Seedance ignore `aspect_ratio` when a frame is given and follow the image. Generate the image at the ratio the video should have, usually `9:16`.
- **Seedance: frames or references, not both.**
- **`last_frame` alone does nothing on most models.** It needs a `first_frame` too.
- Leaving `role` out lets the model decide. Most start from the image, but set `first_frame` explicitly when the user wants the video to start on that image.
