---
title: Nano Banana Pro
name: nano-banana-pro
type: app
version: 0.2.0
requires:
  - GEMINI_API_KEY
description: |
  Generate or edit a single image via Google's Nano Banana Pro
  (Gemini 3 Pro Image). Two actions, one HTTP call each.
---

# Nano Banana Pro

Single-call image generation and editing via the Gemini 3 Pro Image API.
Pass a prompt, get an image.

## Usage

```
/act.generate --prompt "a serene japanese garden" --filename 2026-04-15-garden.png
/act.edit --prompt "Change ONLY: sky to sunset. Keep identical: composition, lighting, foreground." --filename out.png --input_image @image
```

`/act.generate` registers the output path as `@image`, so `/act.edit -i @image ...` chains directly without re-typing the path.

## Per-call defaults

- **Resolution:** `1K` while iterating; pass `--resolution 4K` only when the prompt is locked. 4K costs ~16× the compute.
- **Filename:** `yyyy-mm-dd-hh-mm-ss-<short-name>.png` keeps outputs sorted chronologically.
- **Prompts pass through as-is** — don't sanitize or rewrite. Only rework if the user's wording would force the model to guess.
- **Editing keeps everything else.** Prefix `/act.edit` prompts with *"Change ONLY: &lt;thing&gt;. Keep identical: composition, lighting, palette, background, text."* Otherwise the model drifts.

[Setup, dependencies, troubleshooting →](./requirements.md)

```act.generate
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent -H "x-goog-api-key: $GEMINI_API_KEY" -d '{"contents":[{"parts":[{"text":"{prompt}"}]}],"generationConfig":{"responseModalities":["TEXT","IMAGE"],"imageConfig":{"imageSize":"{resolution}"}}}'
  prompt, -p: string (required) "Image description"
  filename, -f: string (required) "Output path"
  resolution, -r: string "1K, 2K, or 4K" = "1K"
```

```act.generate.response
{@image} = {filename}
{mime} = {Response.body.candidates[0].content.parts[0].inlineData.mimeType}
save: candidates[0].content.parts[0].inlineData.data
decode: base64
to: {filename}
Saved: {filename} ({mime}, {resolution}) — also bound as @image

next: /act.edit -p "Change ONLY: ..." -i @image -f <new-name> · regenerate with a new prompt
```

```act.edit
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent -H "x-goog-api-key: $GEMINI_API_KEY" -d '{"contents":[{"parts":[{"inlineData":{"mimeType":"image/jpeg","data":"{input_image|base64file}"}},{"text":"{prompt}"}]}],"generationConfig":{"responseModalities":["TEXT","IMAGE"],"imageConfig":{"imageSize":"{resolution}"}}}'
  prompt, -p: string (required) "Editing instruction"
  filename, -f: string (required) "Output path"
  input_image, -i: string (required) "Path to source image"
  resolution, -r: string "1K, 2K, or 4K" = "1K"
```

```act.edit.response
{@image} = {filename}
{mime} = {Response.body.candidates[0].content.parts[0].inlineData.mimeType}
save: candidates[0].content.parts[0].inlineData.data
decode: base64
to: {filename}
Saved: {filename} ({mime}, {resolution}, edited from {input_image}) — @image now points here

next: /act.edit -p "Change ONLY: ..." -i @image -f <new-name> · regenerate fresh
```
