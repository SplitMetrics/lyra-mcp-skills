# Adapting a template's prompts to the user's product

A template prompt mixes **craft** (how the shot looks, moves and is paced) with a **subject** (what is in it).
Change the subject. Keep the craft.

## Method

1. Copy the original prompt and mark the subject nouns and claims: product name, app category, the example character's
   identity, on-screen text, the example setting when it is about the product.
2. Replace only those, using the user's details. Keep the same position and level of detail. A product described in
   8 words gets a replacement of about 8 words.
3. Leave camera, lens, lighting, style words, aspect, motion verbs, pacing cues and negative phrasing unchanged.
4. Read it once as the viewer. It should sound as if the producer wrote it for this product.

## Example

Original (image node of a "POV: app launch" template):

> POV shot from the crowd, dozens of raised phones filming a night skyline as a giant glowing **fitness app icon**
> lights up the clouds, on-screen text "POV: the coolest app launch just happened in the sky", handheld, cinematic

Adapted for a budgeting app called Pennywise:

> POV shot from the crowd, dozens of raised phones filming a night skyline as a giant glowing **green piggy-bank
> app icon for Pennywise** lights up the clouds, on-screen text "POV: the coolest app launch just happened in the
> sky", handheld, cinematic

Only the subject changed. The hook text stayed because it is the format. Change hook text only when the user asks or
when it names the example product.

## Rules of thumb

- **Characters:** keep age, framing and expression direction ("talking to camera, surprised"). Change the identity only
  if the user wants a specific persona.
- **On-screen text and scripts:** keep the word count and the punctuation rhythm. Translate them if the user's audience is
  non-English, and set `language_code` on voiceover nodes to match.
- **Real brands:** never put a competitor's or a celebrity's name or likeness into a prompt.
- **Several scenes:** keep the same product wording in every scene, so the product looks identical across clips.
