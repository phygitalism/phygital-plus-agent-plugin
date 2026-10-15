---
name: phygital-which-ai
description: Picks the Phygital+ AI tool for a creative task, writes a description or instruction prompt, and decides whether a reference image is needed. Use when the user asks which model to use, or wants an image, video, text, background removal, or upscale.
---

# Which AI model to Use

Based on the user's request you have to recommend **one** AI tool. A backup only if the first one fails. Do not dump a catalog. Do not mention this file or which AI tool is a fallback.

The model table below is the source of truth.

## Workflow (do not skip)

1. Read the user's task: image, video, text, or finishing.
2. Pick the AI tool from the table below.
3. Write the right prompt type: **description** or **instruction**.
4. Ask for a reference or upload if there is one needed.
5. If it fails — use the fallback in the same row. 

Understand the task first, then AI Tool, then prompt, then reference. Not the other way around.

## How to pick an AI Tool

This table is priority. Use it as-is.

| Task | Recommendation | Fallback |
| --- | --- | --- |
| Advertising (marketing) banner with text or a logo | GPT Image | Nano Banana |
| Strong aesthetic visuals, little or no text | Recraft | GPT Image |
| Icons | GPT Image, version 2 | Recraft |
| Illustrations | Recraft Image | Nano Banana |
| Photorealism: people, product, lifestyle, mockups | Ideogram or Krea | Recraft |
| Text, lettering, or labels on an image | GPT Image | Nano Banana |
| Logo and brand style | Recraft | GPT Image |
| Character or mascot (new poses, emotions, brand look) | Nano Banana | GPT Image |
| Image from a reference, or a series in one style | GPT Image | Nano Banana |
| Edit an existing frame (object, clothes, light, time of day) | Nano Banana | GPT Image |
| Place a product in a scene, or try-on | GPT Image | Nano Banana |
| Video with a person | Hailuo v2.3 (MiniMax Hailuo 2.3) | Seedance (Character or Video reference) |
| Longer clip, scene consistency | Seedance | Hailuo H3 |
| Video with start frame, end frame, or references | Kling | Kling Omni |
| Animate a banner, poster, or object (no person as hero) | Seedance | Kling |
| Talking person or character video | Omnihuman | Gemini Omni |
| Write a script, prompt | Gemini Text, Claude Text, GPT Text | QWEN Text |
| Analyze a document or image and write text | Gemini Text | Claude Text |
| Remove background | Remove Background (Photoroom) | GPT Image v1.5, Transparent background |
| Upscale, add more sharpness and detail | Topaz Upscale Image | Magnific Upscale Image |

## How to write a prompt

Two kinds. The kind follows the task, not habit.

| Task | Prompt type | Example |
| --- | --- | --- |
| Image from scratch | Description | `close-up portrait of a young woman, flowing red hair, soft studio lighting, white background, photorealistic` |
| Edit an existing frame | Instruction | `Change the background to a snowy mountain landscape, keep the person unchanged` |

**Description** — Recraft, Ideogram, Krea. GPT Image and Nano Banana understand it too.  
**Instruction** — edits in GPT Image and Nano Banana.

### Description

From the main idea to the details:

```
[shot type] + [subject] + [how it looks] + [style] + [background] + [keyword for light / quality]
```

```
product photo of a ceramic skincare jar, beige and white, soft daylight,
clean studio background, space for headline, photorealistic, sharp focus
```

- Put the important words first.
- Be specific: not “a woman”, but “a woman in her 30s, short dark hair”.
- Style is the strongest lever: `photorealistic`, `digital illustration`, `minimal`, `cinematic`.
- Don’t write negations like `without background`. Say which background you want.
- Recraft and Krea work better in **English**. GPT Image and Nano Banana take Russian and English.
- For video generation the description prompt can include the timing for scenes in the following AI tools - Gemini Omni, Seedance, Hailuo Minimax, Kling

### Instruction

What to do → to what → how → what to keep:

```
[action] + [object] + [how it should look] + [what to preserve]
```

```
Replace the logo on the shirt with a red star
Replace the background with a white studio and soft backlight. Keep the person and clothing unchanged
```

- One action at a time. Split complex edits into steps.
- Always say what must stay: `keep the face unchanged`, `preserve the composition`.
- Without constraints the model changes more than asked.
- For complex edits and preserving many texts on images GPT Image v2 performs better

If the prompt doesn’t land: shorten it, move the key words to the front, switch to English, or rewrite with Gemini Text / Claude Text / GPT Text.

## Reference

A reference is a photo, product, style, character, or a previous frame. Without it the model invents everything.

Upload one when:
- a product, logo, style or character must stay recognizable;
- style, color, or a character must hold;
- you are editing an existing image — that image is the reference;
- video needs a start frame, end frame, character reference, or video reference.

In the prompt, say what the reference is for: `use the product from the reference`, `keep this character`, `replace the background with the second image`.
