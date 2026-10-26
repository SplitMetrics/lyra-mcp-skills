---
name: lyra-video-ad-generation
description: Base skill for video ads in Lyra. Builds a mobile app video ad end to end (brief, script, scenes, keyframes, clips, voiceover, music, Video Editor assembly) with scene consistency, de-slop and error handling. Use for "a 15-second video ad for my app", "a 30 s / 1 min ad", "a sequence of scenes", "stitch these clips", or a script or storyboard to produce. Holds the core video rules the smaller lyra video skills point to.
---

# lyra-video-ad-generation

The main video skill for Lyra. It holds the core rules for every video ad: script, scenes, music, voiceover, consistency between scenes, de-slop and error handling. Smaller video skills (UGC, hooks, templates) point here for these rules.

Part 1 is shared by every Lyra skill. Part 2 is what's specific to video ads. The Reference library at the end has the details (models, prompts, audio, UI, consistency, de-slop, errors, Meta app ads).

---

# Part 1. [SHARED] How every Lyra skill works

**Deliverable:** a Lyra workflow in which every planned node has been generated. The user opens the canvas and finds the results, the prompts that produced them, and a graph they can edit and rerun for the next app, angle or market.

A full creative always follows these steps, in order:
1. **Context.** Collect the app context and task inputs; make sure you have everything before proposing anything.
2. **Concept and plan.** A very short concept plus a short plan of nodes, models and connections, based on "Best way to build it".
3. **Approval.** The user approves or corrects concept and plan. Nothing is created before this.
4. **Build.** Create the workflow and nodes with the chosen models, write every prompt into the node's prompt field. Nothing runs yet.
5. **Review on canvas.** The user checks the built workflow and prompts and edits anything.
6. **Run options.** Offer ways to run (images first, audio first, everything) with the credits each costs. The user picks one.
7. **Run and report.** Run the picked option, report, offer the next run.
8. **Verify quality.** Check with the user against the quality gates.

## Whose instructions win
1. The user's explicit instructions in this chat.
2. This skill's best practices and defaults.
3. The brand kit: real app facts, claims, value proposition, features, audience.

Best practices are defaults, not rules for the user. When a user's choice will likely hurt the result, follow it and say once in the plan what you'd do instead and why; don't argue twice. When a choice breaks a hard limit (a 60 s clip from a model capped at 30 s), explain and offer the closest build that works.

## Step 1. Context

**App.** Every concept starts from the app. Generic ads come from missing product context, so never fill gaps from imagination.
1. Brand kit given in the chat → use it. Otherwise `list_brand_kits`: one kit → "Make this for {kit name}?"; several → ask which. The user may add or override details in chat.
2. No kit → ask once: send it here, or should I check / connect one of the kits already in your workspace? If the user prefers to type it: app name, what it does, who it's for, the pain it solves, 2–3 features the ad may show, approved claims and offer, forbidden claims, tone, assets (logo, **screenshots only**; never ask for a screen recording).
3. Keep the result as an **app card** for the whole session:
   > App card: Calmly · 5-minute wind-down sessions for sleep · for students 18–25 · pain: can't switch off at night · may show: wind-down timer, sleep sounds · claims: none approved · tone: calm, a bit ironic · assets: logo, 3 screenshots · source: brand kit

**Ask only what only the user knows.** Lyra's own nodes never ask and fill every gap with reasonable assumptions (System Preamble). This skill asks once, in Step 1, only for what can't be assumed (the app, the screenshots, the idea if they have one); for everything else it assumes, states the default in the plan and proceeds.

**Task.** Ask for the inputs in Part 2 → Inputs and defaults; state the defaults instead of asking. Ask for everything missing in one message; never re-ask what the user said. Call `list_workspaces` (credits) and `list_node_configs` (models) now.

## Step 2. Concept and plan

Short: readable in under a minute.

**Concept:** 2–4 lines per deliverable (Part 2 → Concept). N deliverables = N distinct ideas; more takes of one idea is `count` on a node, not more nodes.

**Plan:** name the matching row of Part 2 → Best way to build it in one line, then list the nodes:

| # | Node label | Node · model · key settings | Inputs (role) | Stage |
|---|---|---|---|---|
| 1 | Script (note for the user) | user text | — | — |
| 2 | Character: Mia | image · google/nano-banana-pro · 9:16 · count 1 | — | 1 |
| 3 | Screenshot: timer | upload image | — | 1 |
| 4 | Phone shot: timer | image · google/nano-banana-pro · 9:16 | 2, 3 (references) | 1 |
| 5 | VO: full | audio · elevenlabs/v3 · voice | — | 1 |
| 6 | Music | audio · elevenlabs/music · 20 s · instrumental | — | 1 |
| 7 | Ad: 15 s, 3 shots | video · bytedance/seedance-2.5 · 15 s · 1080p · 9:16 · audio on (ambience only) | 2, 4 (references) | 2 |
| 8 | Video Editor | video_composer | 7, 5, 6 | 3 |

**One clip by default.** A Seedance 2.5 clip holds up to 30 s and several shots ("cut to"), so one clip with the character and UI as references is the default. Split into separate clips only when: the ad is longer than 30 s · a UI moment needs its own insert (Reference D) · scenes need different models or exact frames · one shot failed and must be replaced · the user asks. Say in one line of the plan why you split or don't.

Stages: **1** = sources (images, voiceover, music; fast and cheap) · **2** = built on finished images (video, lip-sync) · **3** = assembly (Video Editor / composer).

## Step 3. Approval
Ask the user to approve or correct. On corrections, show only what changed and ask again. Don't create anything in Lyra before a clear yes.

## Step 4. Build the workflow
1. `create_workflow` (or `use_template`, or the user's own workflow), then `assign_brand_kit`.
2. Write every prompt (Part 2 → Prompt rules): brief, script, image and video prompts, voiceover text, music prompt. The script can go into a user text node (`create_user_text_node`) as a note for the user. A text node does **not** affect generation: never claim it keeps the character consistent or that it feeds other nodes. Consistency comes only from the character image connected as a reference, plus the anchors written into every prompt.
3. Create every node from the plan with its prompt, its model and settings from `list_node_configs`, and a label that says what it's for ("Scene 2: app demo", "VO: hook"), never "Image 1".
4. **Connect the whole graph now** (`connect_nodes`), so the workflow can run on its own with "run all": character → every image and video node that shows the person (reference) · screenshot upload → the phone image · images → video · clips, voiceover, music → Video Editor **in playback order**. Never leave connections "for after the user approves an image". **References** connect before anything is generated (checked in Lyra). **First / last frame** from a generated image needs that image to have a result first (an uploaded image can be a frame right away): on frame models, say in the plan that the frame link is made right after the image runs, and make it automatically before running the clip.
5. Lay out the canvas in run order, left to right, with one Section (frame node) per stage: **Sources** (script note, character, screenshot uploads) → **Images** → **Clips** → **Audio** → **Video Editor**. Put an upload next to the node it feeds, never at the end of the canvas. Don't put the whole graph in one Section.
6. Check `is_valid` and `validation_error` on every node; fix with `update_node` using listed values.

## Step 5. Review on canvas
Tell the user the workflow is built and ready to check in Lyra; ask them to review it and say, or edit on the canvas, what to change. If they edited, `get_workflow` before running and use their prompts as they are now. Never overwrite their edits.

## Step 6. Run options and costs
Offer 2–3 ways to run, each with its credits, so the user sees what a run will spend before it starts. Lyra shows an estimated credit cost per node (the number next to **Run**): read it for every node in the option and report per option. Never guess a price; if the price isn't available to you, ask the user to read it off the nodes.

| Option | Runs | Results | Credits | Time |
|---|---|---|---|---|
| **A. Images first (recommended)** | character + keyframes | N images | {x} | ~1 min |
| B. Images + voiceover + music | stage 1 | N images, VO, track | {x} | ~1 min |
| C. Everything | images → clips → Video Editor | 1 final video | {x} total | ~5–10 min |

Balance: {X} credits.
- Recommend the option that decides the look (character, keyframes). The user picks the best result before more credits go on it.
- Option C has no pause to pick: the graph is already connected, so each stage uses the selected result of the previous one. Keep count 1 on image nodes when the user picks C.
- If the full plan costs more than the balance, say so and offer a cheaper build (fewer ideas, lower count, a cheaper model per Reference A).
- A pick authorises that option only. Each next stage needs a new pick, shown with its credits.

## Step 7. Run and report
1. `run_node` on every node of the picked option whose inputs are ready; keep each `run_uuid` and `variation_uuid`; call `get_run_results`; while `in_progress`, call it again. **Never call run_node on a running node** (second paid run).
2. After images: if a node made several results, ask the user to select one in Lyra before running the next stage (connections already exist; frames and references use the selected result).
3. Run a stage only after its inputs have results; the Video Editor runs last.

Report:
> Calmly · video ad · "2 am": stage 1 done. Open it in Lyra to review, edit any prompt and rerun. Workflow: {link}. Spent {x} credits · balance {X}. Next run: B. clips for the frames you pick, about {x} credits.

- Name failed nodes: label, what it was asked to do, why if known, the fix you suggest.
- When every node has a result, offer 2–3 next steps (new angle on the same concept, other sizes, other languages).
- You can't open result links (they need a Lyra sign-in): don't describe or grade results; send the stage checklist (Part 2 → Quality gates) and ask what works and what doesn't. Refer to nodes by label, never by uuid.

## Step 8. Verify output quality
Check final assets, not only job status: completeness (outputs, order, scenes, CTA) · message (hook, value, claims, language match the brief) · visuals (continuity, correct UI, no major defects) · audio (words, voice, pronunciation, sync, no clipping) · delivery (duration, 9:16, resolution, final render) · variations (intended differences visible) · reusability (inputs editable). Use Part 2 → Quality gates. Suggest fixes only for identified defects, within authorized scope and cost.

## Follow-ups
- One change ("make her older", "faster music"): `update_node` on that node, show the new prompt, give the credits to rerun it and the nodes that depend on it, rerun after a yes.
- A new angle, persona, size or language: hand over to the matching skill.
- Never delete or overwrite results the user may want; add a node or a variation. `delete_node` only when the user asks.

## Runtime failures
See Reference G. In short: validation_error → fix with listed values · run failed → name it, one retry with a changed prompt or model and its credits · in_progress → poll again · not enough credits → stop, show balance and cost, offer cheaper · model refused content → tell the user, don't rephrase around it · file without public link → ask for a link or upload in Lyra.

## Base prompt rules
You write every prompt, and each lives in its node's prompt field, where the user reads, edits and reruns it. Write for the model and for the person.
- Self-contained: no "same as above", no references to the chat.
- Concrete and renderable: things the model can show or say, not marketing concepts.
- The user's own words for the subject and anything they specified stay as given.
- Settings (ratio, duration, resolution, voice, audio) go in node settings, not in the prose.
- English, unless text must appear or be spoken in another language. Exact on-image or spoken text in quotes.
- No placeholders, alternatives or "if" branches in a final prompt.

## Common mistakes (every skill)
| Mistake | Prevention |
|---|---|
| Builds or runs before concept and plan are approved | Steps 1–3 first, always |
| Runs without a picked option, or run_node on a running node | Step 6; poll get_run_results |
| One total cost instead of a cost per run option | Step 6 table, every time |
| Runs over the user's canvas edits | get_workflow before running |
| Invents app features, UI, numbers or reviews | Only the app card; without screenshots keep the screen out of view |
| Swaps the user's explicit choice for a default | Instruction order; mention the trade-off once |
| On-screen text or captions in video prompts | Text in the script, marked for post |
| Re-describes the look in a video prompt with a first frame | Motion only; the frame sets the look |
| Leaves connections for later, so "run all" breaks | Connect the full graph in step 4; frames at the video's ratio |
| Splits a short ad into a node per micro-scene | One clip by default; split only for the listed reasons |
| Voiceover longer than its clip | Reference C pacing; VO ~1 s shorter than the video |
| Values the model doesn't list | Copy from list_node_configs |
| Nodes for takes of one idea, or count for different ideas | Ideas → nodes; takes → count |
| Prompts that point back to the chat | Self-contained prompts |
| Reruns an unchanged prompt as a fix | Change prompt or model; say what changed |
| Describes or grades results it can't see | Ask the user with the stage checklist |
| A forbidden claim from the Brand Kit, a political line or a dig at a competitor | Brand safety (Reference B): forbidden claims enforced absolutely |

---

# Part 2. Video ad generation: what's specific

## Description
Creates multi-scene video ads for mobile app paid ads in Lyra, SplitMetrics' node-based creative canvas, leaving a generated, reusable workflow on the canvas.
Use when the user asks for "a 15-second video ad for my app", "a 30 s / 1 minute ad", "a sequence of scenes", "stitch these clips together", or hands over a script or storyboard.
Works from a Lyra brand kit or app details in chat, app screenshots, a script, a reference ad (through the Creative Analysis node), or an existing workflow.
Not for: a UGC creator video (the UGC skill), a hook only (the hook skill), static images (the image skills). Those skills use this skill's rules for scripts, scenes, audio, consistency and de-slop.

## Quality bar
Stops the scroll in the first 1.5–3 s, shows only real app features and real screens, reads as one consistent film (same person, place, light), sounds natural (voice not rushed, music without vocals under speech), no AI-slop look, and works with sound off.

## Inputs and defaults
**Required (ask if missing):** the app (brand kit or app card) · what the ad is about or the idea, if the user has one · app screenshots, if the ad shows the app.
**Defaults (don't ask; state them in the plan):** Reels / TikTok · **9:16** (16:9 only if asked) · **15 s** · English · voiceover on · music on (instrumental) · **Seedance 2.5** recommended for real people, **Seedance 2.0** for illustration / UI-heavy · **1080p** (set it: Seedance defaults to 720p) · in the plan, always ask which model and quality the user prefers, with your recommendation and why · **sound on** at generation (mute in the Video Editor if needed) · Nano Banana Pro for keyframes · 1 result per node.
**Recommended first run:** A. Images first.

## Concept
A good concept names one pain the audience recognises, in their words, and shows the app solving it with one real feature, one beat per scene, CTA at the end. It also says **why** it should work: the user is trying to learn what works and why (System Preamble). Write it as the hypothesis + beats:
> Video, 15 s. We believe students who can't switch off at night will start a trial because a relatable 2 am POV shows the wind-down timer working. 0–3 s hook: "POV: 2 am, brain won't stop", tired face in phone light · 3–11 s she starts the timer (real screenshot), shoulders drop · 11–15 s eyes close, CTA "Try it tonight". Voice: tired, warm. Music: soft lo-fi, instrumental.

Brief, structure, scene fields, timing and montage rules: Reference B.

## Best way to build it
The user's explicit choices override any column. Model choice and price rules: Reference A.

| If the request has | Best approach (graph) | Models | Why |
|---|---|---|---|
| **Nothing specific (default)**, real people, ≤30 s | character image (+ phone image with the real screenshot if the app is shown) → **one** multi-shot clip ≤30 s with both as **references** → VO + music → Video Editor | seedance-2.5 · nano-banana-pro | fewest nodes, one consistent look; Seedance holds identity from references |
| UGC / creator talking to camera | hand over to the UGC skill; if it runs here: Seedance 2.5 only, never Kling or Veo | seedance-2.5 | Seedance is clearly better at people |
| 60 s or longer than 30 s | full script → two 30 s parts (or split text) with the same character and style references → two clips → Video Editor | seedance-2.5 | no model makes 60 s in one clip |
| Illustration, cartoon, stylized, moving text | keyframe per scene → clip with image as **frame** | seedance-2.0 | holds style; frames keep the picture exact |
| The app's UI on screen | user screenshots → phone image (character + screenshot) → short clip, or UI insert laid over in Video Editor (Reference D) | seedance-2.0 (frames) or seedance-2.5 (refs) | models break UI in long busy clips |
| Exact start and end frame, complex multi-shot | first + last frame per clip | kling-v3-video (Pro) | precise frame control (not for UGC) |
| Very short product / nature shot, ≤8 s | references → one clip | veo-3.1 | realism; max 8 s, used rarely |
| Copy a movement from a reference video | character image + motion video | kling-v3-motion-control | motion transfer |
| A reference ad to adapt | Upload Video → Creative Analysis → script (written by you from its output) → as default row | as default row | keeps the mechanism, not the identity |
| Many drafts / tight budget | draft model for tries, rerun the winner on the recommended model | seedance-2.0-fast, veo-3.1-lite | spend on the winner only |

## Prompt rules

| Node | Write | Avoid |
|---|---|---|
| Image (keyframe) | concept in 2–3 words → subject → action → setting → framing → light → color/lens → mood → style → visible details → exclusions in plain words. ≤500 chars. The moment it shows (start of the motion if used as first frame). | abstract adjectives; text; drawing the user's logo or UI |
| Video with a frame | motion only: what moves, camera move, pacing, end state; spoken line in quotes if the model speaks | re-describing the look; on-screen text |
| Sound line (every clip) | what the clip should sound like. Separate music: "Sound: only natural ambience and action sounds (…); no music, no soundtrack, no score". Separate voiceover and nobody speaks on camera: add "no speech, no voice" | leaving sound unspecified (the model adds its own music) |
| Video with references | who is who first ("[Image1] is the character's face and outfit, [Image2] is the phone screen"), then scene and motion; must-keep identity early | mixing frames and references on Seedance; unassigned references |
| Video without image | full scene as for an image, then the motion | more than one scene per clip unless multi-shot with "cut to" |
| Voiceover | exact words only, one continuous text, numbers as spoken, sized to the video at the right pace, ~1 s shorter than the video | stage directions, labels, brackets; lines longer than the clip |
| Music | genre, era, BPM, instruments, energy curve (hook hit → build → CTA resolve), "instrumental only, no vocals" when anyone speaks, length = video + ~5 s | artist names; vocals under speech; length equal to the ad |

**Prompt length:** one shot 60–150 words (sweet spot 80–120); image-to-video 40–60% shorter; over ~300 words later instructions get ignored. Non-negotiables first.

**The 5 rules that matter most here:** (1) one scene, one main action, one camera move; (2) anchors repeated word for word in every prompt (Reference E); (3) no generated text or UI, ever; (4) style from the brief, phone-real for UGC, no "stunning 8K" adjectives; (5) plan the end frame of every clip.

Full prompt craft (nine blocks, video formula, camera, light, materials, physics, phrase bank, worked example, checklist): Reference F.

## Common mistakes (this skill)
| Mistake | Prevention |
|---|---|
| Image connected as first/last frame on Seedance 2.5 (error) | Seedance 2.5 = references only |
| Character's face changes between scenes | Character image first, passed as reference to every scene; same model for all clips |
| UI on the phone garbled or invented | Real screenshots only; phone image + UI insert (Reference D) |
| Clips come out 720p | Set 1080p explicitly on Seedance |
| Music fades out early / has vocals under the voice | Generate ~5 s longer, instrumental, trim in Video Editor |
| Voiceover runs over the video | Words for (video − 1 s) at a calm pace; shorten text or faster voice, never regenerate video |
| Artist name in the music brief (error) | Describe genre, tempo, mood |
| Sound turned off at generation | Keep sound on; mute in Video Editor |
| Kling or Veo offered for UGC | Seedance 2.5 always for UGC |
| Claims a text brief/script node keeps the character consistent | Text nodes are notes; consistency = character image as reference + anchors |
| Clip comes with its own music next to the generated track | Sound line in every clip prompt: ambience only, no music |
| Whole graph in one Section, uploads at the end | One Section per stage, left to right, uploads next to the node they feed |
| Clips out of order in the Video Editor | Connect in playback order; after results check and set `startMs` |
| Price guessed | Read the price on the nodes; per option |
| Too many references "because the model allows it" | Small, purposeful set; video/audio refs only on request |
| A 1-minute clip promised | Hard limit; two halves or split, joined in Video Editor |
| Same prompt rerun after a failure | Change the prompt, reference or model; after two fails rewrite the scene |

## Quality gates
Claude can't see the results, so after each stage it sends the stage checklist and asks the user what they like and what they don't (Reference H has the full slop list):
- **Stage 1 (images):** same person in every keyframe · hands and phone natural · screen exactly like the screenshot · style right (phone-real for UGC) · nothing plastic or over-glossy.
- **Stage 2 (clips):** face and outfit stable within and across clips · no morphing, flicker, warped hands · natural motion · UI correct while tapping · ends on the planned frame.
- **Stage 3 (final):** hook in 1.5–3 s · scenes in order, CTA at the end · voiceover not rushed, ends before the video, app name pronounced right · music instrumental under speech, trimmed with a fade · 9:16, 1080p, planned length · text inside the safe zone.
Pass = the user is happy with every point that applies. Fix only what fails, within the agreed budget, with its credits.

---

# Reference library

## A. Models, prices and settings

Always call `list_node_configs` first: it is the live source of models, durations, resolutions and limits.

**How to offer a model.** Recommend **Seedance 2.5** (real people) or **Seedance 2.0** (illustration, stylized, UI/UX). Say plainly: "This is the more expensive option, but it gives the best quality for this video." Give the price per node and per run option (step 6). Always ask which model the user prefers before building (in the plan, step 2–3). Seedance 2.5 and 2.0 always come before any Kling. **UGC and real people talking: always Seedance 2.5, never Kling or Veo**; if the user wants cheaper, say the quality on faces and speech will drop and let them decide.

**Cheaper ladder** (one step at a time, show the new price, skip models bad at the task): Seedance 2.5 / 2.0 → Veo 3.1 (only clips ≤8 s) → Kling 4.0 (only once it is in Lyra's list) → Kling 3.0 Pro → Kling 3.0 Standard → Veo 3.1 Fast → Seedance 1.5 Pro (short lip-sync only) → Seedance 2.0 Fast → Kling 2.6 → Kling 2.5 Turbo Pro → Veo 3.1 Lite / Seedance 2.0 Mini. Drafts first, final later: test ideas on a draft model, rerun only the winner on the recommended one.

**Task → model:** real people, UGC → Seedance 2.5 · illustration, cartoon, UI/UX, moving text → Seedance 2.0 · product / nature realism, ≤8 s → Veo 3.1 · exact first + last frame, complex multi-shot → Kling 3.0 Pro · motion transfer → Kling 3.0 Motion Control Pro (Standard cheaper) · same character with video and voice refs → Kling Video 3.0 Omni · short lip-sync with synced audio → Seedance 1.5 Pro · campaign character consistency → Runway Gen-4.5 · strong physics → Sora 2 Pro (Sora 2 for drafts) · edit an existing video by chat → Gemini Omni · talking head (face + audio) → veed/fabric-1.0 (720p).

**Model cards (Lyra, Oct 2026)**
- **Seedance 2.5** (flagship, real people): 4–30 s or `adaptive` (fits length to the dialogue/script); the only Lyra model up to 30 s. **References only: never first/last frame (the fields exist but error out).** Technical max 30 images / 10 videos / 10 audio, never pushed (see reference policy). Refs are named [Image1], [Image2]… in the prompt. Default 720p → set 1080p. Angle and details can shift.
- **Seedance 2.0** (illustration, UI/UX): 4–15 s, `-1` = intelligent duration. Image as **frame** (start/end; picture kept exactly) or **reference** (looser), never both in one node. Rule: image with UI or any text → frame; no text and exactness not important → reference; unsure → ask. Technical max (per list_node_configs) 9 image / 3 video / 3 audio refs, never pushed. Default 720p → set 1080p. Fast and Mini: cheaper, max 720p, drafts.
- **Veo 3.1:** 4/6/8 s (max 8), references (≤3) like the others, 9:16 default. Best realism, product, nature. Used rarely, only very short clips. Fast / Lite: cheaper, frames only.
- **Kling 3.0 Pro / Standard:** 3–15 s, first frame and/or last frame only, no references. Multi-shot, complex scenes, cartoon/stylized; weaker than Seedance on faces and human motion; cheaper than Seedance.
- **Kling 3.0 Motion Control Pro / Standard:** image + reference video 3–30 s (`character_orientation: video` up to 30 s, `image` up to 10 s). References with clear hands, head and body; avoid complex angles and blocked limbs.
- **Kling Video 3.0 Omni:** up to 7 reference images (4 with a video ref), first/last frame, 3–15 s; native audio can't be combined with a reference video.
- **Kling 2.6:** 5/10 s, first frame, native audio. **Kling 2.5 Turbo Pro:** 5/10 s, fast, cheap, no audio. **Seedance 1.5 Pro:** 5/10 s, lip-sync, dialogue.
- **Sora 2 / 2 Pro:** 4/8/12 s, first frame only. **Runway Gen-4.5:** 5/10 s, first frame.
- **Kling 4.0** (launched 16 Oct 2026): 3–30 s, up to 10 keyframes, stereo audio, cheaper than Seedance. Not in Lyra yet; offer only when list_node_configs shows it, and never ahead of Seedance for people.
- **Images:** Nano Banana Pro for finals (text, UI, people, realism, sharp), Nano Banana 2 for drafts.

**Settings in the Video Generation node:** model → seconds (or adaptive) → aspect ratio 9:16 (confirm) → quality 1080p (tell the user it keeps details and faces best; they may choose otherwise) → sound **on** → how images connect (frame or reference, by model).

**Reference policy.** Limits are a technical maximum, not a target: many extras make the model unpredictable. Never tell the user how many references a model takes or suggest adding many. Default references = the images you generated (character, keyframe, product, UI). Write the montage yourself in the script; a video reference only when the user asks or the brief requires copying an edit (it works only as an editing reference). An audio reference only when the user brings a track (the video is fitted to it).

**Longer than a model allows (e.g. 1 minute).** Say honestly no model makes it in one clip (30 s Seedance 2.5, 15 s others, 8 s Veo). Offer: (A) two 30 s videos from the same idea and image, joined in the Video Editor; or (B) the full 60 s script split into two 30 s parts (check each part's length), the same reference(s) or a few style references for both, two videos in one style, joined in the Video Editor. Same model, character, style anchor, voice and music in both halves. Always tell the user first that this is possible and how.

**Price.** Lyra shows an estimated credit cost per node (number with the coin icon next to Run). Read it for every node that will run and warn the user before running, per node and per option. Never guess. After a run, report what it cost (credits before − after, `list_workspaces`).

**Text nodes.** Claude writes brief, script, voiceover script and music brief itself into user text nodes (the AI text nodes can't be created with the tools). Exception: **Creative Analysis** (`pattern_import`): when the user brings a reference video and the node is available, use Upload Video → Creative Analysis and write the script from its output.

**Video Editor** = the `video_composer` node (no credits). Connect clips, voiceover, music and images in playback order at build time (it lays clips out in connection order; to reorder, disconnect and reconnect). After the clips have results, read the composition with get_workflow and check the order; if it's wrong, set each clip's `startMs` with update_node. Pick a transition, run. Its edit lives in `video_rendering_composition` (get_workflow / update_node): `canvas` (fps, width, height, durationMs) and `tracks` (video / audio layers; clips with `startMs`, `durationMs`, `trim.sourceStartMs/sourceEndMs`, `volume`, `position`; track `volume`, `muted`). Trims, music volume under the voice (e.g. 0.25), muting a clip's own sound and overlay layers (a UI insert above the main video) are set there. Read the existing composition first and change only what's needed; captions are added in the editor UI.

## B. Brief, script and scenes

**Brief header:** app · platform · aspect ratio · length · language · voiceover · music · audience (Brand Kit).
**Hypothesis:** "We believe [audience] will [action] because [concept] addresses [pain] through [mechanism]" (a whole concept, not one variable).
**Then:** angle (≤100 chars) · summary (≤200) · hook mechanism · hook line (≤80; on-screen version ≤40) · value proposition (≤150) · proof point (Brand Kit or user only) · tone · offer · CTA (≤30) · format · image style · palette · must include / exclude · reference notes.
**Competitor references:** keep the idea (concept, structure, hook, format); never their logo, colors, visuals or text.
**Claims:** never invented, pricing included: no price, discount or offer that isn't listed in the Brand Kit or given by the user. AI people are never presented as real customers; testimonials only from real reviews. No body before/after for health or weight-loss apps. **Brand safety** (Lyra's System Preamble): nothing that could damage the advertiser's reputation; no profanity, no political statements, no unsubstantiated health claims, no competitor disparagement. The Brand Kit's **forbidden claims are enforced absolutely**: never in a line, a caption, a visual or the VO; if the user asks for one, say once that it is on the kit's forbidden list and leave it out unless the user confirms the kit is outdated.

**Structures:** problem → agitate → demo → CTA (default) · hook → solution → social proof → CTA · POV / relatable moment → app as the fix → CTA · before/after of the situation · single continuous shot. Scene types: hook, problem, solution, social proof, offer, cta, logo end card. Brand only at the end.

**Scene fields:** number + type · duration · visual direction (like an image brief: subject, setting, framing, angle, light, palette, mood, style, texture; no text; style per brief) · motion (one main action) · VO line · on-screen text ≤40 chars with position and animation, **marked for post** · music/SFX cue · transition · consistency (source scene, must match, allowed to change) · asset (screenshot, logo) · generation note · end state. Mark the hook scenes; give them the most detail.

**Timing:** scene durations add up exactly to the total. A beat on screen is 1.5–5 s; a clip is generated at a length the model allows and trimmed in the Video Editor. UI clips ~4 s. Hook within ~1.5 s (Reels/TikTok) to 3 s (Feed). 5 s clip beats: 0–1 establish · 1–3 action · 3–4 peak · 4–5 resolve.

**Montage:** written by Claude in the script (shots, order, length, cuts). Default: the shots live inside one clip with explicit "cut to" markers. Separate clips joined in the Video Editor only for the reasons in Step 2 (over 30 s, UI insert, different models or frames, a failed shot, the user asks).

**Keyframe:** the frame that starts the scene (start of the motion when used as first frame; the peak goes in as last frame on frame models). Generated at the video's ratio.

## C. Audio: voiceover and music

**Voiceover.** Write it yourself (from the script and Brand Kit) into a "VO script" text node, then an audio voiceover node (`elevenlabs/v3`, voice from `list_voices`).
- Exact spoken words only: no scene labels, timestamps, "Hook:", "VO:", "[pause]", "(upbeat)", brackets, asterisks, slashes, markdown, separator dashes. Allowed: comma, period, ellipsis, ?, !. Pause = comma or ellipsis.
- One continuous text joining the per-scene lines; rhythm shows the pace; the words carry the tone (never describe it).
- No invented claims. The user's direction wins (tone, length, focus, language, style).
- **Pace (English):** 2.5 words/s conversational (default), 3.0 fast/energetic, 2.0 slow/emotional.
- **Always ~1 s shorter than the video** (15 s video → 14 s VO ≈ 35 words, calm, not rushed). Check the real audio length after generating. Too long → cut words or a faster voice; never regenerate the video for the voiceover.
- **Voice:** filter by language, accent, gender, age, use case; pick by the voice description to fit the audience and tone; play a sample; reuse the same saved voice across a campaign.
- **Languages:** same script, new VO script "Speak in <language>" + its own voiceover node with a voice in that language; run in parallel; match each in the Video Editor; check each language's length (translations run longer).

**Music.** Write the music brief from the script's mood; music node `elevenlabs/music` by default (Lyria 3 / Lyria 3 Pro if asked).
- Prompt: genre, era, BPM, key/mood shift, instruments, sync points (hit on the hook, lift on the reveal, resolve on the CTA), hook in the first 2 s, fade-out. Example: "Dark-to-hopeful cinematic electronic underscore, 110 BPM, D minor shifting to F major at the CTA; pulsing synth bass, metallic percussion, risers and impacts on the press, resolving into clean uplifting synth chords. Instrumental only, polished modern mix, smooth 1 s fade-out."
- **Anyone speaks in the video → instrumental only, no vocals, no singing, no humming.** Keep `force_instrumental` on.
- **Length = video + ~5 s** (10 s video → 15 s track, `music_length_ms`), trimmed in the Video Editor, because the fade starts early.
- **No artist names** (they can error). Check the script too.

**Clip sound:** sound stays on at generation, but say in every clip prompt what it should be. With separate music: "only natural ambience and action sounds, no music, no soundtrack, no score". With a separate voiceover and nobody speaking on camera: also "no speech". Otherwise the model adds its own music and the clip audio can't be used next to the track.

**Mix in the Video Editor:** voice is the main sound; music clearly under it; trim music to the video and fade on the last frame; mute or lower clip sound that clashes (sound stays on at generation).

## D. App UI in the video

- Use as little UI in video as possible; **never invent UI**. Seedance 2.5 / 2.0 often break UI in long busy clips.
- If the screen is needed, ask: "Please send screenshots of the app screens you want in the ad." Screenshots only, never a screen recording. No screenshots → screen dark, blurred or turned away.
- **Phone image:** a second image node: "the same character holds the phone close to camera, screen facing us"; connect the main character image and the user's screenshot. Run it, then ask the user: "Is the UI exactly like your screenshot (layout, text, colors, logo)? Same person, hands and light?" Rerun the image before any video if not.
- **Way 1, one clip** (about 10 s, max 15 s): connect the character image and the phone image to the video node (Seedance 2.5 as references; Seedance 2.0 as frames, because there is text; Kling as first/last frame).
- **Way 2, UI insert:** if the UI came out bad but the rest is fine, make a short clip from the phone image (screen + finger tapping or scrolling, a few seconds) and lay it over the broken part in the Video Editor (layer above, trimmed to the moment). Plan Way 2 from the start when the UI moment is long or important.
- Show one key screen or "aha moment", not a full walkthrough. Ask after every UI result; fix only the broken part.

## E. Consistency between scenes

- **Anchors** defined once and repeated word for word in every prompt: character (age, face, hair, skin, outfit, accessories), environment (place, time of day, light direction and color), style (palette from the Brand Kit, grading, look, lens), product/phone (device, case, screen from the screenshot).
- Per scene: **source scene** (earliest that set the element) · **must match** · **allowed to change**.
- One image model for all keyframes, **one video model for the whole ad**, one ratio and resolution.
- **Character first, connected everywhere:** the character image is generated in stage 1 and **connected at build time as a reference to every image and video node that shows the person**, so the graph stays consistent even with "run all". If the user wants to choose between several characters, they run stage 1 alone and select one; the connections are already there. A text brief alone never makes images consistent. Two characters for A/B: one script, two character images, two videos.
- **By model:** Seedance 2.5 references (roles per [ImageN]); Seedance 2.0 frame for exactness / UI, reference for looser; Kling first/last frame; Omni references + video ref.
- Frames pin to the **selected** result; reconnect after regenerating a keyframe.
- **Hand-off:** write each clip's end state; the next scene starts from it; keep motion direction across cuts; match cuts, whip pans and cuts on action hide small differences.
- **When it breaks:** strengthen the reference and move the anchor earlier, shorten the prompt, reduce motion; a reference leaking light or framing → state its job or drop it; two failed tries → simplify the scene in the script.

## F. Prompt craft

**Image, nine blocks:** concept (2–3 words) → main subject and what it does → context → lighting → color and technique (palette, lens) → mood → composition (front, back, sides) → style (if you name a director, also describe the look) → visible details.

**Video formula:** SUBJECT → ACTION → CAMERA → ENVIRONMENT → LIGHT → MATERIALS → PACE → EMOTION → ENDING → CONSTRAINTS. A framework, not a template; one coherent direction, not a stack of tags.
- **Action:** one clear change of state + a few natural secondary moves. Weak: "a woman with a phone". Strong: "she glances at the notification, her eyebrows lift, and a small smile spreads as she turns the phone toward the camera."
- **Camera:** a move for a reason: static (control), slow push-in (intimacy), pull-out (scale), tracking (follow), orbit, pan, tilt, crane, handheld (UGC presence), whip pan (energy), rack focus. Subject motion and camera motion stated separately.
- **Framing:** eye level (natural), low (power), high (distance), overhead (geometry), three-quarter (volume), profile (silhouette); shot size; subject placement in the central 65% of 9:16.
- **Light:** one main approach + 1–2 supporting (soft diffused, hard directional, Rembrandt, backlight, rim, golden hour, blue hour, volumetric).
- **Palette:** one dominant, from the Brand Kit, kept across scenes; shift only if the shift is the story.
- **Materials:** how they react to light and motion (glass reflections stay coherent; heavy fabric doesn't float).
- **Physics:** cause and result (fabric settles, hair follows the head, a thumb really presses the screen).
- **Pace:** slow deliberate, fluid, sharp, energetic, weightless; matched to the scene.
- **Lens and focus:** 24mm wide, 35mm environmental, 50mm natural, 85mm tight, macro; say what is sharp at each key moment. Guidance, not guarantees.
- **Finish:** clean digital (precise detail, natural skin) or analog (fine grain, halation); UGC = phone camera, natural light, slight handheld.
- **Ending:** plan the final frame (eye contact, product centered, phone to camera, brief hold).
- **Image-to-video:** bring the image to life, don't replace it; preserve composition, face, outfit, light; limited movement (blink, breath, gaze, small head turn, restrained push-in); "no new objects, no scene transition, no wardrobe changes".
- **Rules:** one scene one idea · action beats adjectives · camera move needs a reason · complexity fits duration · light interacts with materials · no conflicting styles · constraints short and prioritized · clarity beats length · plan the final frame · consistency before spectacle when identity, product or UI matter.
- **Phrase bank:** smooth: "fluid motion, slow deliberate movement, graceful follow-through" · identity: "preserved facial features, consistent wardrobe, stable character appearance" · realism: "natural inertia, believable body mechanics" · product: "precision product cinematography, controlled reflections, macro detail" · light: "soft directional lighting, subtle rim light" · UGC: "shot on phone front camera, natural window light, slight handheld wobble, casual, unpolished" · ending: "controlled final pose, brief visual hold".
- **Pre-generation check:** subject clear · identity preserved where needed · main action explicit · camera defined · composition and location fit · light and palette coherent · hands, hair, objects plausible · pace fits duration · final frame intentional · key unwanted changes constrained · settings match the model · nothing contradictory.

## G. Errors

| Problem | Fix |
|---|---|
| Seedance 2.5 errors with an image | It was a frame → reconnect as reference |
| Seedance rejects inputs | Frames and references mixed → pick one |
| Video ignores the script or image | Connect both to the video node |
| Video stays on the old image | Keyframe regenerated → reconnect it |
| Duration rejected | Use a listed value (model first, then seconds) |
| Clip is 720p | Set 1080p; Seedance 2.0 Fast/Mini can't do 1080p |
| Music node error | Artist name → describe genre, tempo, mood |
| Vocals under the voice | Instrumental only, force_instrumental on |
| Music fades early | Generate ~5 s longer, trim |
| Voiceover too long | Shorten to (video − 1 s) or faster voice |
| Clip audio clashes | Mute/lower in Video Editor |
| Clips in wrong order | Disconnect and reconnect in playback order |
| UI garbled | Phone image + UI insert (D) |
| Face drifts | E → "When it breaks" |
| Text appears in video | Remove it from the prompt; text goes on in the Video Editor |
| Longer than the model allows | Two halves or split script (A) |
| Kling 4.0 asked for | Not in Lyra yet; offer Seedance 2.5 |
| Two failed tries | Rewrite the scene in the script; if the image confuses the model, use it as a reference instead of a frame (where allowed); never rerun unchanged |

## H. De-slop

**Prevent:** style from the brief (UGC phone-real, not "premium cinematic") · real skin ("natural skin texture, visible pores"), never "flawless" · lived-in places with 2–3 specific details · no "stunning, 8K, masterpiece, hyper-realistic" · simple hand actions, one object in hand, hands visibly touching · faces frontal or three-quarter, one person in focus · one action, one camera move, physical cause and result · no generated text, UI or logos · a short constraints line. Images: "text overlays, captions, watermark, blurry, low quality, distorted faces, extra limbs". Video: "text generation, on-screen captions, morphing faces, identity drift, outfit change, flickering, jitter, warped hands, unstable geometry, sudden camera jumps" + 1–3 scene risks. If the model has no negative field, put a short "Avoid …" line at the end of the prompt.

**Slop signs:** faces (plastic skin, identity change, morphing, dead eyes) · hands (extra or merged fingers, passing through objects, floating phone) · motion (rubber limbs, sliding feet, floaty movement, jitter) · text/UI (garbled letters, invented or changing screens, warped logo) · scene (flicker, objects popping, melting background, warped lines) · look (over-glossy, over-saturated, stock vibe, wrong style) · audio (robotic or rushed voice, wrong app name, VO over the video, vocals under speech, abrupt music end) · edit (mismatched cuts, wrong order, CTA cut off, text outside the safe zone).

## I. Brand Kit

Every brief, script, prompt, voice and music direction is built on the Brand Kit: audience first, then product, value, tone, colors, claims, forbidden claims (enforced absolutely), must include / exclude. Source order: the chat → the kit connected to the workspace / workflow → ask (send it, or check and connect one of the workspace's kits). The user's chat request always wins; the kit fills everything they didn't say; mention a conflict in one line.

## J. App ads on Meta (what works)

- Vertical 9:16 video first (Reels and Stories carry most app-install volume); 15–30 s; a hook that shows the app working, not a logo.
- 3–5 hook variants per concept; test one variable at a time; 2–3 new concepts a month (creatives fatigue in days to weeks); Advantage+ app campaigns need several different concepts.
- Formats that convert: UGC style (phone-shot, person to camera) beats polished brand video on cold traffic · problem → agitate → demo → CTA ending on the key screen · real people using the app · one simple key screen, not a walkthrough · before/after of the situation · specific, plausible social proof from the Brand Kit.
- Hook in ~1.5 s (Reels/TikTok), by 3 s (Feed); no dead air; no logo intro. Hook rate (3-s views ÷ impressions) >30% good, <20% weak.
- Length: Reels 6–30 s (15–20 s safe), Stories 6–15 s, Feed 15–30 s (4:5 or 1:1 as a later variant).
- Sound off: most viewers watch muted → captions for all speech, key message also on screen, same words as the VO; large, high-contrast, one consistent place; added in the Video Editor.
- Safe zone: subject and text in the central 65% of 9:16, away from the top bar and the bottom caption/CTA area.
- CTA: a concrete offer ("Try free for 7 days") beats "Download now"; in VO and on screen in the last 2–3 s.
- Judge creatives by trials, payers and day-7 retention, not only cost per install; give a test 2–3 weeks.
