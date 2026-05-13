# Nano Banana Pro for String 🍌

Google's Nano Banana Pro (Gemini 3 Pro Image) image generation and editing — exposed as two String actions. Pass a prompt, get an image. Edit by reference, not by copy-paste.

## What you get

- **`@image` chaining.** `/act.generate` registers the output path as `@image`; `/act.edit -i @image ...` iterates without re-typing the file path.
- **Single-call simplicity.** No SDK, no Python helper. The actions are direct REST calls to Gemini, plus `save → decode base64 → to <path>` to write the bytes.
- **Iteration-aware defaults.** `1K` resolution for prototyping, `4K` only when the prompt is locked (~16× the compute).

## Install

```
string main '/install --app ./nano-banana-pro/string.md'
string app:nano-banana-pro '/set $GEMINI_API_KEY = "AIza..."'
string '/open app:nano-banana-pro'
```

Get a key at [Google AI Studio](https://aistudio.google.com/apikey). `/set` must run from the app's own session — keys are scoped per-app and never leak across apps or from the shell. See [requirements.md](./requirements.md) for the full setup and common errors.

## What it looks like

```
$ string app:nano-banana-pro '/act.generate -p "a small red ceramic lobster on a wooden desk, soft natural light, shallow depth of field" -f /tmp/lobster.jpg'

Saved: /tmp/lobster.jpg (image/jpeg, 1K)

next: /act.edit -p "Change ONLY: ..." -i @image -f <new-name> · regenerate with a new prompt
```

Iterate on the same composition — `@image` points at what you just made:

```
$ string app:nano-banana-pro '/act.edit -p "Change ONLY: time of day to sunset, warm orange light through the window. Keep identical: lobster, desk, books, mug, composition, framing." -i @image -f /tmp/lobster-sunset.jpg'

Saved: /tmp/lobster-sunset.jpg (image/jpeg, 1K, edited from /tmp/lobster.jpg)

next: /act.edit -p "Change ONLY: ..." -i @image -f <new-name> · regenerate fresh
```

`@image` rebinds to the *new* file after each edit, so a sequence of edits chains naturally:

```
/act.generate -p "..."           → registers @image = file1.jpg
/act.edit -i @image -p "..."     → registers @image = file2.jpg
/act.edit -i @image -p "..."     → registers @image = file3.jpg
```

## Actions

| Action | What |
|---|---|
| `/act.generate -p "..." -f path.jpg` | Generate from scratch. |
| `/act.edit -p "Change ONLY: ..." -i src.jpg -f new.jpg` | Edit an existing image. |

Both accept `-r 1K|2K|4K`. Default `1K`.

## Notes

- One-time setup, API key handling, troubleshooting: [requirements.md](./requirements.md)
- The Gemini API returns JPEG bytes regardless of your output filename's extension — `.jpg` matches the actual mime type, `.png` works too (bytes are still valid JPEG).
- Adapted from [@steipete](https://clawhub.ai/steipete/nano-banana-pro)'s `nano-banana-pro` skill. The original ships as a Codex/Claude Code skill with a 167-line Python helper (`uv run` + `google-genai` + `pillow`). This SFMD port drops the Python entirely — just `curl` + `jq` + `base64`, all of which ship with String.

---

> 📘 **Cookbook reference.** A two-action app that demonstrates: `save → decode base64 → to <path>` binary pipeline · `{@image}` shortcut binding for self-chaining actions · `{input_image|base64file}` template helper · single-file design when journey is linear.
