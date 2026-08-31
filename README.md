# Infographic Studio

> A single-file, zero-dependency, mobile-first web app that turns repository URLs, documentation, or pasted text into professional, one-shot product infographics.

[![CI](https://github.com/benneberg/infographic-studio/actions/workflows/ci.yml/badge.svg)](https://github.com/benneberg/infographic-studio/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-0-2ea44f.svg)](https://github.com/benneberg/infographic-studio)
[![Single File](https://img.shields.io/badge/Architecture-Single%20File%20HTML-blue.svg)](https://github.com/benneberg/infographic-studio)

---

## Features

- **BYOK** — Groq (fast inference) and OpenRouter, OpenAI-compatible `/chat/completions`.
  Keys live in `localStorage` only and are sent straight to the chosen provider.
  Includes key testing and model discovery via `/models`.
- **Projects CRUD** — create, edit, duplicate and delete projects; each keeps its sources,
  style options and the last 20 generations (with restore).
- **Sources** — GitHub repo (metadata, stars, language mix, top-level tree, README via the
  GitHub API, optional token), any web/docs URL (in-browser fetch with an optional reader-proxy
  fallback for CORS-blocked pages), or pasted text.
- **One-shot generation** — the model returns a strict JSON infographic spec (headline, value
  prop, honest highlights, capabilities, how-it-works flow, tech stack, audience, CTA), which is
  rendered deterministically to SVG. Layout auto-scales to fit the chosen canvas.
- **Formats & themes** — portrait 4:5, square, story 9:16, wide 16:9; midnight / daylight /
  ember / forest. Changing them re-renders instantly without another model call.
- **Export** — 2× PNG, SVG, or copy the spec JSON.

## Usage

```bash
# any static server works
npx serve apps/infographic-studio
# or just: open apps/infographic-studio/index.html
```

1. **Settings** → paste a Groq or OpenRouter key, pick the active provider and model, save.
2. **Projects → + New** → name the product, add sources.
3. **Generate** → review in **Result**, tweak format/theme in **Style**, export PNG/SVG.

## Notes

- Groq default model: `llama-3.3-70b-versatile`; OpenRouter default: `openai/gpt-4o-mini`.
- Source material is truncated to roughly 38k characters, split across sources.
- The prompt forbids invented metrics — highlights must be grounded in the supplied material.
