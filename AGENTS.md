# AGENTS.md

> Any update to this file should be reflected in the human-facing README.md.

## Project Overview

Terminal Scroller is a single-file web tool (`index.html`) that emulates a terminal scrolling through long output. Text prints word by word, starting at a readable pace and accelerating exponentially to emphasize output volume.

## Architecture

Everything lives in one self-contained `index.html` — HTML structure, CSS themes, and all JavaScript. No build tools, no dependencies, no framework. Open it in a browser.

### Key Sections in index.html

- **CSS themes** (lines 10–176): 20 terminal themes defined as CSS custom property blocks on `data-theme` attributes. The default (Claude Code) uses `:root`.
- **HTML** (lines 388–461): Container with header, textarea, controls (Start/Stop/Reset/Export GIF), settings row (cols/rows/duration/title), terminal window with titlebar and body, status indicators.
- **Animation engine** (lines 528–643): `computeDelayParams` computes the pacing curve. Two-phase approach for tight time budgets: readable prefix at 60ms, then exponential decay. `printNext` handles word-by-word output with batching when delays drop below 4ms.
- **GIF export** (lines 649–869): Loads gif.js from CDN (fetched as text, worker inlined as blob URL to avoid cross-origin issues). Renders frames to an offscreen canvas with scroll support, samples at max 200 frames.

### Pacing Curve

The delay function is `initial * (min/initial)^progress` where progress goes 0→1. Default auto mode: 80ms initial, 2ms min, ratio 40. When a duration is set and the budget is tight, a readable prefix (first ~5% of words at 60ms) is followed by steeper decay. Word batching kicks in below 4ms to overcome setTimeout floor limitations.

### Adding a Theme

Add a `[data-theme="name"]` CSS block with all custom properties, and a corresponding `<option>` in the theme `<select>`. Properties: `--bg`, `--text`, `--accent`, `--term-bg`, `--term-border`, `--titlebar`, `--input-bg`, `--input-border`, `--btn-bg`, `--btn-text`, `--btn-hover`, `--glow`.

## Testing

Serve locally and test with Playwright MCP:

```
python3 -m http.server 8919
# Then use browser_navigate, browser_snapshot, browser_click, etc.
```

Verify: themes render, scroll animation works with auto-scroll, Stop preserves text, Reset clears, GIF exports without console errors, cols/rows/duration/title settings apply correctly.
