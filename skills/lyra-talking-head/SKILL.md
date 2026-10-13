---
name: lyra-talking-head
description: Make a lip-synced talking-head video in Lyra, a face image speaking a voiceover, or produce a standalone voiceover or music track. Use when the user wants "a presenter saying…", "UGC-style testimonial", "spokesperson video", "make this character talk", "voiceover for this script", or "background music".
---

# lyra-talking-head

A talking head is three nodes: **face image + voiceover audio → talking head video**.

## Steps

1. **Workflow.** Reuse one or `create_workflow`.
2. **Face.** `create_image_generation_node` showing a portrait: front-facing, mouth closed or neutral, face clearly lit, head and shoulders, at the video's aspect ratio. Run it and have the user pick a result.
3. **Script.** Use the user's text as is. If they only have a topic, draft the script and confirm it with them. At a natural pace, ~2.5 words is about 1 second.
4. **Voice.** `list_voices` (use `search` for gender, accent or style). Offer 2–3 voices with their names, or use the one the user named.
5. **Voiceover node.** `create_audio_voiceover_generation_node` with `prompt` = the script, `model_name` from `list_node_configs` (audio, `audio_voiceover_generation`; currently `elevenlabs/v3`), `voice` = voice uuid, and `language_code` if the script isn't English.
6. **Talking-head node.** `create_talking_head_node` with the model from `preferred_models_by_action.talking_head` (currently `veed/fabric-1.0`). Then `connect_nodes` the face image → talking head and the voiceover → talking head, both **without `role`**.
7. **Confirm, then run in order.** Run the voiceover (~10–20 s), let the user listen if they want to, then run the talking head (~2.5–4 min). `get_run_results` repeatedly until `completed`.

## Voiceover / music only

- Voiceover alone: steps 3–5 and run it.
- Music: `create_audio_music_generation_node`. `elevenlabs/music` takes `music_length_ms` (5 000–300 000) and `force_instrumental` (default true). The Lyria models take only the prompt. Lyria 3 Pro accepts song structure tags (`[Verse]`, `[Chorus]`) and timestamps in the prompt. Describe the genre, instruments, mood and tempo.

## Gotchas

- `stability` on `elevenlabs/v3` accepts only `0`, `0.5` or `1`. Other values make the node invalid.
- The face must look like one person, facing the camera. Profiles, several faces, hands over the mouth, or a stylised face with no clear mouth all lip-sync badly. If a result is bad, regenerate the face before you change anything else.
- The audio sets the video length. A long script makes a long, expensive video, so keep it under ~30 s unless the user asks for more.
- To change the words, edit the voiceover and rerun both nodes. Rerunning only the talking head reuses the old audio.
