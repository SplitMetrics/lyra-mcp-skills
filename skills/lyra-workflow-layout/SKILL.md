---
name: lyra-workflow-layout
description: Use whenever a Lyra workflow is created or edited, and for "tidy up the canvas", "sign the sections", "make the workflow clear", "add notes for the user", "the canvas is confusing", "lay it out like the templates". Lays out nodes in sections in run order with STEP notes, the team's color code and clear labels.
---

# lyra-workflow-layout

Helper skill for every Lyra skill. The other skills decide **what** to build; this one decides **how it reads on the canvas**. Every workflow Claude builds or changes is laid out by this file.

**The goal.** Anyone who opens the workflow (the user tomorrow, a teammate, a client) understands in ten seconds what it makes, in which order it runs, and where to change something by hand. A canvas of unlabelled nodes in one row is a black box. A canvas split into colored sections, each with its own small STEP note, reads like a recipe, the way Lyra's own templates do: first the idea, then what the user brought, then the heroes, the scenes, the clips, and the final video.

**Who decides what.** The creative skill decides which sections its creative needs (its "Layout" lines). This file decides how they are placed, numbered, explained, colored, labelled and checked. The user's explicit wishes win over both.

**This is a way of thinking, not a script.** Use judgment when a canvas needs something different, and say why in one line. Fixed: every node in a section, one STEP note per section, steps in run order, the color code, labels a person understands, no empty or stray nodes, and Lyra's technical limits below.

---

## The house style: how Lyra's templates are laid out

Lyra's templates (Creators workspace, Templates) are made by the creative producers, and they all follow one pattern:

- **A reading path.** The script on the left, the work sections in the middle stacked top to bottom, the Video Editor on the right. The eye goes left to right, and top to bottom inside a column.
- **A small STEP sticky note above every section**, in the section's color, numbered in the order the user goes through it:
  - "STEP 1 · Check the script"
  - "STEP 2 · Insert a screenshot/video of your app."
  - "STEP 3 · Generate Image & Video"
  - "STEP 4 · Generate Music · You can create your own music and combine it with the video in the Video Editor"
  - "STEP 5 · Combine scenes"
- **Each note is short:** the step number, a 2 to 5 word action, at most one helper sentence. Plain instructions, no explanations of the tech. One note per section, never one big note with all the steps.
- **Color by kind of step:** lime for the script, pink for the user's own app assets, violet for every generation, rose for the final assembly.
- **One section per kind of work, with its nodes side by side**, flowing left to right inside it.

The templates also have slips that Claude's canvases never repeat: "Gerate", duplicate or missing step numbers, a STEP 0, "STEP 3&4", notes in the wrong color, "(copy)" and "Note (" labels, the editor called Composer in one note and Video Editor in the next, a "Generate Music" note in a workflow with no music node, empty leftover notes and empty upload nodes.

---

## What the tools can do (Lyra, Nov 2026)

Re-check at the start of a build: tools change. Everything below is done with the MCP tools; a browser is only needed to look at the result.

| Need | How | Notes |
|---|---|---|
| Where nodes land | Every `create_*_node` | A new node goes 150 px right of the rightmost node, level with its top. **Nodes created in run order already form a left-to-right row in run order.** |
| Where nodes are, how big | `get_workflow`: each node's `box` (`x`, `y`, `width`, `height`) | Canvas pixels, y grows downwards. A node Lyra hasn't drawn yet has an estimated size (320 wide; a 9:16 image or video node ~809 tall). |
| A section | `create_frame_node` (node_uuids, label, background_color) | Drawn around its nodes: 40 px padding on every side plus a 40 px title bar on top. If it would cover other nodes or sections, its nodes are moved into it right of the canvas. The label is the section title. |
| Section color | `background_color` on `create_frame_node`, later `update_frame` | A named swatch (see the color code). Rename a section with `update_node`'s label. |
| Add nodes to a section later | `update_node` on the frame: `{"node_uuids": [the full list]}` | The whole list. Then `set_node_positions` on the new nodes (inside the section's row) so the frame redraws around all of them. |
| A STEP sticky note | `create_note_node` (text, label, background_color, `above_node_uuid`) | With `above_node_uuid` = the section's frame, the note sits 40 px above it, left-aligned. 240×200, enough for a STEP note. Edit it later (and notes from a template) with `update_note`. |
| Exact positions | `set_node_positions` ([{node_uuid, x, y}]) | Top-left corners. **Moving a frame carries its nodes.** Moving nodes inside a frame redraws the frame around them, growing or shrinking it. **Notes move only themselves**, so place notes after the sections are final, or move them in the same call. |
| Tidy automatically | `organize_nodes` | Follows the connections left to right; separate parts (an unconnected script, a lone upload) are tiled in reading order, left to right then top to bottom, top-aligned; a note right above a section (up to 160 px) moves with it; frames keep their nodes. It only rearranges what is there: it can't build the column grid below. |
| Run one step | The **Run** button on a section's top right corner | Runs every node of that section. |
| Remove an empty or stray node | `delete_node` | Only nodes that hold nothing and feed nothing (an empty upload), or Claude's own leftovers; say which in the report. Anything with content: ask first. |

**An older Lyra server.** No `set_node_positions` in the tool list means `organize_nodes` is the old one: it puts unconnected parts last (the script at the far end) and throws notes to the edges. Then never call it on a canvas with notes or a script section; rely on creation order instead (Build order below).

**Lyra's own Organize nodes button** (bottom toolbar, four squares) still works the old way. Never press it, and tell a user who asks that it scatters the STEP notes; offer `organize_nodes` instead.

**The open page.** Keep the workflow page closed while the MCP tools edit it: an open page can save an older canvas over the changes. Open it again only to look.

---

## Layout: the reading path

Place the sections on a grid, like the templates:

```
 column L          column M1                  column M2                 column R
┌───────────┐   ┌─────────────────────┐    ┌─────────────────────┐   ┌──────────────┐
│ note ST1  │   │ note ST2            │    │ note ST6            │   │ note last    │
│ SCRIPT    │   │ YOUR APP (pink)     │    │ KEYFRAMES           │   │ VIDEO EDITOR │
│ (lime)    │   ├─────────────────────┤    ├─────────────────────┤   │ (rose)       │
└───────────┘   │ note ST3            │    │ note ST7            │   └──────────────┘
                │ CAST                │    │ CLIPS               │
                ├─────────────────────┤    ├─────────────────────┤
                │ note ST4  PLACES    │    │ END CARD · MUSIC    │
                └─────────────────────┘    └─────────────────────┘
```

- **Column L:** the idea: script, brief, concept card (lime), at the top.
- **Middle columns:** the steps in order, top to bottom, then the next column: the user's own materials first (pink), then the heroes, the places, keyframes, clips, end card, voiceover, music.
- **Sections linked scene by scene share a column.** When each node of one section feeds the matching node of the next (keyframe N → clip N, phone shot N → clip N, hook keyframe → hook clip, ad → its sizes), stack the two sections in the same column, the source on top: scene N's clip sits right under scene N's keyframe, at the same x. Each link is then a short drop to the node below. Never split such a pair across columns (keyframes at the bottom of one column, clips at the top of the next): every link would cross the canvas diagonally, through other sections and notes. Balance the columns with the sections around the pair (Cast, Places, music) instead.
- **Column R:** the Video Editor or the final outputs (rose), at the height of the clips.
- **Each section:** its STEP note on top (left-aligned with the section, ~40 px gap), then the section with its nodes in one row, left to right in their own order (keyframe → clip, clip 1 → clip 4). More than 5–6 nodes → two sections ("Clips 1–3", "Clips 4–6").
- **Spacing:** 40 px between nodes inside a section, ~140 px between a section and the next note below it, ~150 px between columns, nothing overlapping.

**When the grid is needed.** A short creative (up to ~5 sections) reads fine as one row: build it in run order (Build order) and it already lands Script → … → Video Editor, left to right, with each note on its section. Use the column grid when the row gets too long to read, and to slot nodes added later into their step.

**Placing the grid with `set_node_positions`:**
- Read the frames' boxes from `get_workflow`. Each column is as wide as its widest frame.
- Move **the frames** (their nodes follow): a frame's y = the note's y + 240 (200 note + 40 gap); the next note in the column starts ~140 px below the frame's bottom.
- Nodes of a section in one row: x = frame x + 40 + i × (node width + 40), y = frame y + 80. After moving nodes this way, the frame redraws itself to hug them.
- A scene-by-scene pair (keyframes over clips): give both frames the same x and node i of both rows the same x, so each source sits straight above its target.
- Then the notes: each at (frame x, frame y − 240), in the same call or created now with `above_node_uuid`.

---

## Sections

1. **One section per step of work, in run order.** Every node sits in exactly one section. Nothing floats outside, nothing is kept "just in case".
2. **One STEP note per section**, above it, in the section's color. Numbering STEP 1, 2, 3… with no gaps, no repeats, no STEP 0, no "3&4", no "2.1".
3. **The section title is short**, because the note carries the step: "Story script", "Your app & style", "Cast", "Keyframes", "Video clips 1–3", "Music", "Video Editor".
4. **Size:** 1 to 6 nodes per section. A section with one node only for the script, the Video Editor, the music or a single upload.
5. **Never** the whole graph in one section, never a section per node, never an "unused" section: empty nodes are removed (see the tools table), real content gets a real step.

### Color code

Pass the swatch name as `background_color` (`lime`, `pink`, `violet`, `rose`, `slate`); Lyra stores it as the 50% transparent hex shown, which is what a workflow from a template has. The note and its section have the same color; the same kind of step has the same color across the canvas.

| Swatch | Hex | Kind of section | What goes in |
|---|---|---|---|
| Lime | `#bef26480` | The idea, text to read and check | Script, brief, concept card, story, hook bank, carousel script, Text Split |
| Pink | `#f9a8d480` | The user's own materials | Screenshots, logo, app icon, packshot, end card or app video, product photo, style reference, brand assets |
| Violet | `#c4b5fd80` | Generation | Characters, places, keyframes, phone shots, clips, voiceover, music, Image Resize, Image Translation |
| Rose | `#fda4af80` | The result | The Video Editor; the final ads or sizes the user downloads |
| Slate | `#cbd5e180` | Reference kept for analysis (nothing runs) | A competitor's ad, a teardown, prompt tips, when the creative needs them |

### STEP notes

A note has two parts, and Lyra shows both: the **title** (the `label`, drawn in the note's header bar) and the **text** (the body). The step number goes in the title only:

```
title:  STEP 2
text:   **Your app & style**

        Added: Lull Home screen, Lull logo and the art style reference. Swap them to change the app.
```

- **Title** (`label`): exactly `STEP N`. Nothing else: no action, no section name, never empty or a default "Note (…)".
- **Text**: line 1 the action in bold, 2 to 5 words ("Check the script", "Our heroes", "Generate keyframes", "Generate video", "Generate music", "Combine scenes"); a blank line; then one sentence: what's in the section or what the user can do ("You can…", "Edit a prompt and press Run.", "Added: …").
- **Never repeat `STEP N` in the text**: the header already shows it, and a second "STEP 2" right under it reads as a slip. When fixing a template's notes, move a "STEP N" line from the text into the title the same way.
- For the user's materials, the note lists what was added ("Added: logo, 3 screenshots").
- In the user's language (English when the canvas and the audience are English); the words the user sees in Lyra: section, node, prompt, Run, Video Editor, Edit Video, Render. Never tool names, uuids or connection roles.
- Short: it must fit 240×200 without scrolling.

**Examples** (video; written as title · action · sentence): "STEP 1 · Check the story script · The idea, every line of dialogue and the timing of all clips." · "STEP 3 · Our heroes · Emma and Dan. Every scene uses them as references." · "STEP 5 · Generate keyframes · One image per clip. Edit a prompt and press Run." · "STEP 7 · Generate music · You can create your own music and combine it with the video in the Video Editor." · "STEP 9 · Combine scenes · Open Edit Video to trim and add captions, then Render."

### Standard sections by creative

The creative skill's own Layout lines say which sections it needs; these are the defaults.

| Creative | Sections in reading order (color) |
|---|---|
| Video ad (base, UGC, short drama, UI motion) | Script (lime) → Your app (pink, when there are uploads) → Cast (violet, when there is a person) → Places / Keyframes → Clips → Voiceover → Music (violet) → Video Editor (rose) |
| Hooks | Hook bank and script (lime) → Your app (pink) → one section per hook with its keyframe and clip (violet) → shared body, voiceover, music → the Video Editors (rose) |
| Short drama | Story (lime) → Your app & style (pink) → Cast → Places → Phone shots → Clips per act → End card → Music (violet) → Video Editor (rose) |
| Static ads | Concept (lime) → Your app (pink) → Persona / product (violet) → Ads (violet, or rose when last) → Sizes and languages (rose) |
| Carousel | Story and Text Split (lime) → Your app and persona → Cards in one row in card order (rose when last) |
| Competitor copy | Competitor reference, analysis only (slate, first, its own note "Reference · not connected") → the normal set |
| Variations, multiplier, audience variations | The original layout kept in each variation; notes and labels updated where the person, setting or angle changed |
| Resize, translation added later | A new section with the next STEP number, placed after the last step |
| Brand Kit starter | Brand assets (pink) · Brand Kit notes: app card, pain map, open questions (slate) |

---

## Node labels

- **Say what it is and where it belongs:** "Scene 2 · app demo · keyframe", "Scene 2 · app demo · clip", "Character · Mia", "Screenshot · timer screen", "Logo", "VO · full script", "Music · lo-fi", "Hook A · clap + pain · clip", "Ad C1 · 2 am POV · 4:5", "Card 3 · fix · 4:5".
- **Default labels only for the single node of its kind:** "Video Script", "Music Brief", "Voiceover Script", "Video Editor". The final assembly node is always called **Video Editor**.
- **No step numbers in node labels** (the note carries the step), no "(copy)", no "Image 1", no "Upload Image", no empty label.
- **One pattern per kind of node**; the same scene, card or hook number in its keyframe, clip and caption.

---

## Build order

1. **Plan.** The plan table (the base skill's Step 2) gets a Section column; the approval message lists the steps in one line ("STEP 1 Script → STEP 2 Your app → STEP 3 Cast → … → Video Editor").
2. **Nodes in run order**, section by section: the script first, then the user's uploads, the heroes, keyframes, clips, music, the Video Editor last. Connections as the creative skill says. Remove Claude's leftovers and empty nodes.
3. **Sections:** `create_frame_node` per section, with its label and swatch, in run order.
4. **Positions** (only for the column grid or nodes added out of order): `set_node_positions` on the frames, then on nodes inside a section if their row needs it.
5. **Notes:** `create_note_node` per section with `label` = `STEP N`, the text without the step number (STEP notes), `above_node_uuid` = its frame and the same swatch, once the sections stand where they stay.
6. **Look** (when a browser signed in to Lyra is available): open the workflow, Fit view, screenshot, check the reading path and that nothing overlaps, close the page. Without a browser, check the boxes from `get_workflow` instead.
7. **Check** (below), fix, check again.
8. **Report** the steps in the chat with the same words as the notes, plus what the user does next.

**Later changes to a canvas Claude built** (a new hook, a size, a language): new nodes, then their section and note with the next STEP number, then `set_node_positions` to put the section after the last step and its note above it. `organize_nodes` is fine here too when the result should simply follow the connections.

---

## Templates, variations and the user's own workflows

**A workflow made from a template (`use_template`).** Keep the template's own layout: its notes and sections were placed by the producers. Fix the notes with `update_note` (typos, continuous numbers, colors from the code, "Video Editor", notes that no longer match their nodes). Never call `organize_nodes` there (nor the button). Nodes Claude adds get their own note and section, placed at the right step with `set_node_positions`.

**Variations** (`create_variation`) copy the whole canvas with its layout. Keep it; update the notes and labels that name what changed. Each variation has its own nodes: lay out each one.

**The user's own workflow.** Don't rearrange or delete without asking; add Claude's nodes in new noted sections and offer the tidy-up in one line.

---

## Check before handing over

`get_workflow` (the boxes) and, when a browser is available, a screenshot:
- [ ] Every node that isn't a frame or a note appears in exactly one frame's `node_uuids`.
- [ ] One note per section, above it, same color; STEP 1 to N continuous, in reading order.
- [ ] Each note's title is `STEP N` and its text starts with the action, with no second "STEP N".
- [ ] Sections linked scene by scene are stacked in one column, scene N above scene N; no link crosses into another column.
- [ ] The script section is first (left, top); the Video Editor last (right).
- [ ] No empty nodes, no "unused" section, no one big note with all the steps.
- [ ] No "(copy)", "Image 1", "Upload Image" or empty labels; the editor is "Video Editor".
- [ ] Nothing overlaps; every section hugs its nodes (no big empty frames).
- [ ] All connections, prompts and results the same as before the layout.

## Common mistakes

- One big note with every step instead of a note per section.
- Lyra's Organize nodes button, or the old `organize_nodes`, on a canvas with notes: the notes end up on the edges and the script last.
- Notes placed before the sections were moved: `set_node_positions` doesn't carry notes.
- Nodes created out of run order and left there: the row reads in creation order, so fix it with `set_node_positions`.
- A text node used as a STEP note: it is a real input; use `create_note_node`.
- An "unused" section full of empty uploads.
- Editing through MCP while the workflow page is open (the page saves an older canvas).
- STEP 0, gaps, repeats, "3&4"; a note that says one thing while the nodes do another.
- Step numbers in node labels; titles in a language the user doesn't use.
- "STEP N" in both the note's title and its text: the header shows it twice.
- Keyframes in one column and their clips in the next: every keyframe → clip link crosses the canvas.
