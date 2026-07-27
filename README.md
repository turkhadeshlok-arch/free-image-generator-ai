# Shl0ka AI

## What it is

Shl0ka AI is a free, no-signup AI image generator that runs entirely in the browser. It's a single HTML file — no backend, no database, no API key to configure, and no build step. You open the file (or host it as a static page) and it works immediately.

The interface lets you:

- Type a text prompt and generate an image from it
- Pick a model from a live list fetched from Pollinations
- Choose an aspect ratio (1:1, 3:4, 4:3, 16:9)
- Set or randomize a seed, so a specific result can be reproduced later
- Toggle "enhance prompt" to let a model expand/improve your wording before generation
- Toggle "private" to keep the generation out of Pollinations' public feed
- Browse a session-only history/gallery of everything generated in the current tab
- Download the generated image or copy the prompt back out

Because everything runs client-side, there's nothing to deploy on a server beyond serving the static file itself.

## How it works

Every generation is a single `GET` request to Pollinations' public image endpoint:

```
https://image.pollinations.ai/prompt/{encoded-prompt}?width=&height=&model=&seed=&nologo=true
```

Query parameters used:

| Param | Purpose |
|---|---|
| `width`, `height` | Output image dimensions, set by the aspect ratio buttons |
| `model` | Which generation model to use (populated from `https://image.pollinations.ai/models`) |
| `seed` | Controls reproducibility — same prompt + same seed = same image |
| `nologo` | Suppresses Pollinations' watermark |
| `enhance` | Optional — routes the prompt through a model that improves the wording first |
| `private` | Optional — excludes the generation from Pollinations' public feed |

When you click **Generate**, the app builds this URL and loads it as an `<img>` src. There's no intermediate server: the browser talks to Pollinations directly, and the resulting image is rendered as soon as it loads.

State management is intentionally simple:

- The model dropdown is populated once on page load via a `fetch` to Pollinations' `/models` endpoint, with a hardcoded fallback list if that request fails.
- The seed field is auto-filled with a random number if left blank, so every generation is reproducible even if you didn't set one yourself.
- Every successful generation is pushed into an in-memory JavaScript array (capped at the most recent 24) and rendered as a thumbnail grid below the main stage. Clicking a thumbnail reloads that prompt/seed combination back into the main view.
- The "download" button attempts to fetch the image as a blob and trigger a save dialog; if the response can't be fetched as a blob (e.g. CORS restrictions), it falls back to opening the image in a new tab so it can be saved manually.

Nothing is written to disk, a database, or `localStorage` — closing or refreshing the tab clears all history.

## Tech

- **HTML / CSS / vanilla JavaScript** — no frameworks (React, Vue, etc.), no bundler, no package.json, no build step
- **Fonts** loaded from Google Fonts via CDN link tags
- **[Pollinations.ai](https://pollinations.ai)** — the underlying image generation API; both the `/prompt/` generation endpoint and the `/models` listing endpoint are used
- Runs in any modern browser (Chrome, Firefox, Safari, Edge) with no server-side runtime required

## Limitations

- **Depends entirely on Pollinations' uptime and rate limits.** Since there's no self-hosted model or fallback provider, generation speed and availability track whatever Pollinations' public infrastructure is experiencing at the time — expect slower responses or occasional failures during high load.
- **No accounts, no persistence.** There's no login system and no server-side storage. History exists only in the current browser tab's memory and disappears on refresh or close.
- **No content moderation layer.** The app doesn't filter, review, or restrict prompts/outputs beyond whatever Pollinations enforces on its own models — it's a thin client over their API, not a moderated product.
- **CORS-dependent downloads.** The download button relies on being able to fetch the generated image as a blob; if Pollinations' CORS headers change or a specific model/response doesn't support it, the app falls back to opening the image in a new tab instead of a direct download.
- **No offline mode.** Since generation, model listing, and font loading all depend on external network requests, the app doesn't function without an internet connection.
- **Single global session, single device.** There's no mechanism to sync or share history across devices/browsers/tabs.
