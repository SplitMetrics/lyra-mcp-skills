---
name: lyra-video-ad
description: Build a multi-scene video ad in Lyra end to end — script, one keyframe and clip per scene, optional voiceover and music, joined by the video composer into one video. Use when the user asks for "a 15-second ad", "a video ad for my app", "a sequence of scenes", "stitch these clips together", or hands over a script or storyboard.
---

# lyra-video-ad

The shape is: **scenes → keyframe image per scene → video clip per scene (first_frame) → composer**, with an optional voiceover and music track fed into the composer.

## Plan (no credits)

1. **Brief.** Collect the product, audience, tone, length (default 15 s), aspect ratio (default 9:16) and call to action.
2. **Script.** Write 3–5 scenes. For each one write the on-screen visual, the motion and the duration. Add the voiceover line if there is one. Keep the scene durations within the video model's duration options and make them add up to the target length. Show the script and get it approved before you create any node.
   - Write the script yourself. To keep it on the canvas next to the scenes, add it with `create_user_text_node`.
     That node holds text you wrote and runs no model.
3. **Consistency.** Pick one image model, one video model and one style phrase (lighting, palette, lens) that every scene prompt shares. The scenes only vary in subject and action. Mixing models across scenes makes the ad look like separate clips.
4. **Cost.** Count the runs: N images + N clips + voiceover + music. The composer is free. State the counts, the models, the estimated time (clips run in parallel, so ~3–6 min) and the workspace credits. Get a yes.

## Build

1. `create_workflow` titled after the ad.
2. Per scene: `create_image_generation_node` (keyframe, at the ad's aspect ratio) and `create_video_generation_node` (motion prompt, the scene's duration). Label them `Scene 1 – image`, `Scene 1 – video`, and so on.
3. Optional audio: `create_audio_voiceover_generation_node` (full voiceover script, voice from `list_voices`), `create_audio_music_generation_node` (instrumental, length ≈ ad length). If you add a voiceover, set `generate_audio: false` on the clips so their native audio doesn't clash with it.
4. `create_video_composer_node` with an optional `transition_effect` (`crossfade` and `fade` are safe defaults).
5. Optionally, `create_frame_node` around each scene's nodes to keep the canvas readable.

## Run (order matters)

1. `run_node` **every** keyframe image and the audio nodes, then collect them all with `get_run_results`.
2. Show the keyframes. Regenerate any off-brief scene now, because an image costs far less than a clip.
3. `connect_nodes` each image → its video with `role: first_frame` (this needs the image to be finished).
4. `run_node` **every** clip, then collect them all. Video runs take minutes, so call `get_run_results` repeatedly.
5. `connect_nodes` each clip → composer **in playback order** (scene 1 first), then voiceover → composer and music → composer.
6. `run_node` the composer → `get_run_results`. Give the user the final `url`, plus the per-scene clip links.

## Gotchas

- The composer joins inputs in the order they were connected. To reorder, `disconnect_nodes` the clips and connect them again in the right order.
- Rerunning one scene's clip is enough to fix that scene. Then rerun only the composer. Don't rerun the whole graph.
- If a keyframe is regenerated after it was connected, connect it again (`first_frame`). The video stays pinned to the old image until you do.
- Voiceover length should fit the total clip length. If it runs over, shorten the script rather than stretching the clips.
