# AGENTS.md

> Any update to this file should be reflected in the human-facing README.md.

## Project Overview

Terminal Scroller is a single-file web tool (`index.html`) that emulates a terminal scrolling through long output. Text prints word by word, starting at a readable pace and accelerating exponentially to emphasize output volume.

## Architecture

Everything lives in one self-contained `index.html` — HTML structure, CSS themes, and all JavaScript. No build tools, no dependencies, no framework. Open it in a browser.

### Key Sections in index.html

- **CSS themes**: 20 terminal themes defined as CSS custom property blocks on `data-theme` attributes. The default (Claude Code) uses `:root`. Properties: `--bg`, `--text`, `--accent`, `--term-bg`, `--term-border`, `--titlebar`, `--input-bg`, `--input-border`, `--btn-bg`, `--btn-text`, `--btn-hover`, `--glow`.
- **HTML**: Container with header, textarea, controls (Start/Stop/Reset/Export GIF), settings row (speed mode radio, cols/rows/duration/title), terminal window with titlebar and body, status indicators.
- **Animation engine**: `computeDelayParams` computes the pacing curve, `getDelay` returns the delay for a given word index, `printNext` handles word-by-word output with batching.
- **GIF export**: Loads gif.js from CDN (fetched as text, worker inlined as blob URL to avoid cross-origin issues). Renders frames to an offscreen canvas with scroll support. Uses adaptive frame sampling for word-by-word fidelity at the start.

### Speed Modes

A radio toggle selects between two modes:

- **Accelerating** (default): exponential decay curve `initial * (min/initial)^progress`. Default auto mode: 80ms initial, 2ms min, ratio 40. With a duration set and tight budget, uses two-phase pacing (readable prefix + steep decay).
- **Constant**: uniform delay for every word. With a duration set, delay = target / word count. Without a duration, defaults to 80ms/word.

### Pacing Curve (Accelerating Mode)

**Two-phase pacing** for tight time budgets (duration set, many words): when the computed initial delay would drop below 60ms, the system switches to a readable prefix (first ~5% of words at 60ms) followed by steeper exponential decay for the remaining words.

**Word batching**: when the computed delay drops below 4ms (the browser's setTimeout floor), multiple words are printed per tick to maintain the intended pace. Batch size = ceil(4ms / delay).

### GIF Frame Sampling

The GIF export uses adaptive frame sampling instead of fixed-step sampling. Delays are accumulated word by word; a frame is emitted when accumulated delay exceeds 20ms. This gives one-frame-per-word fidelity at the readable start and batched frames during the fast tail. If total frames exceed 300, the tail is downsampled while keeping early frames intact.

### Adding a Theme

Add a `[data-theme="name"]` CSS block with all custom properties, and a corresponding `<option>` in the theme `<select>`.

## Testing

Serve locally and test with Playwright MCP:

```
python3 -m http.server 8919
# Then use browser_navigate, browser_snapshot, browser_click, etc.
```

Verify: themes render, scroll animation works with auto-scroll, Stop preserves text, Reset clears, GIF exports without console errors, cols/rows/duration/title settings apply correctly. For duration testing, use `browser_evaluate` with a Promise-based timer to measure actual elapsed time vs target.
