# AGENTS.md

> Any update to this file should be reflected in the human-facing README.md.

## Project Overview

Terminal Scroller is a single-file web tool (`index.html`) that emulates a terminal scrolling through long output. Text prints word by word, starting at a readable pace and accelerating exponentially to emphasize output volume.

## Architecture

Everything lives in one self-contained `index.html` — HTML structure, CSS themes, and all JavaScript. No build tools, no dependencies, no framework. Open it in a browser.

### Key Sections in index.html

- **CSS themes**: 20 terminal themes defined as CSS custom property blocks on `data-theme` attributes. The default (Claude Code) uses `:root`. Properties: `--bg`, `--text`, `--accent`, `--term-bg`, `--term-border`, `--titlebar`, `--input-bg`, `--input-border`, `--btn-bg`, `--btn-text`, `--btn-hover`, `--glow`.
- **HTML**: Container with header, textarea, controls (Start/Stop/Reset/Export GIF), settings row (speed mode radio, cols/rows/duration/title), terminal window with titlebar and body, status indicators.
- **Animation engine**: Tunables live in the `CONFIG` object at the top of the script. `computeDelayParams` computes the pacing curve, `getDelay` returns the delay for a given word index, `printNext` prints words against a wall-clock schedule (catch-up batching, one DOM fragment per tick) so throttled timers don't break the Duration target. Run state lives in a `session` object; Stop pauses it and Start resumes it (button label flips to "Resume") only when the text, speed mode, and duration are unchanged, otherwise it restarts. Under `prefers-reduced-motion`, Start dumps the full output instantly.
- **GIF export**: Loads gif.js from CDN (fetched as text, verified against pinned SHA-384 digests via SubtleCrypto before execution; worker inlined as blob URL to avoid cross-origin issues). Renders frames to an offscreen canvas with scroll support, using an incremental word-wrap layout (`createLayoutState`) that advances token by token so N frames cost O(tokens) instead of O(frames x tokens); lines scrolled permanently out of view are trimmed (a `trimmed` count keeps the scroll offset anchored) to bound memory on long exports. Uses adaptive frame sampling for word-by-word fidelity at the start; `maxFrames` scales down with canvas area to bound memory (roughly 150 MB of RGBA frames). After encoding, the worker pool is terminated and frame buffers released.

### Speed Modes

A radio toggle selects between two modes:

- **Accelerating** (default): exponential decay curve `initial * (min/initial)^progress`. Default auto mode: 80ms initial, 2ms min, ratio 40. With a duration set and tight budget, uses two-phase pacing (readable prefix + steep decay).
- **Constant**: uniform delay for every word. With a duration set, delay = target / word count. Without a duration, defaults to 80ms/word.

### Pacing Curve (Accelerating Mode)

**Two-phase pacing** for tight time budgets (duration set, many words): when the computed initial delay would drop below 60ms, the system switches to a readable prefix (first ~5% of words at 60ms) followed by steeper exponential decay for the remaining words.

**Word scheduling**: printing runs against a wall-clock schedule. Each tick prints every word whose cumulative scheduled delay has already elapsed (inserted as one DOM fragment with a single scroll), then sleeps until the next word is due (minimum `CONFIG.tickFloor` ms, the browser's setTimeout floor). Timer throttling and jitter self-correct instead of accumulating drift.

### Settings Persistence

Speed mode, cols, rows, duration, and title are saved to localStorage (`terminal-scroller-settings`) on change and restored on load; values are validated/clamped when loaded. Numeric inputs are also clamped at read time via `getCols`/`getRows`/`getDurationS`. The theme is persisted separately (`terminal-scroller-theme`).

### GIF Frame Sampling

The GIF export uses adaptive frame sampling instead of fixed-step sampling. Delays are accumulated word by word; a frame is emitted when accumulated delay exceeds 20ms. This gives one-frame-per-word fidelity at the readable start and batched frames during the fast tail. If total frames exceed `maxFrames` (scaled by canvas area, capped at 300), the tail is downsampled while keeping early frames intact.

### Accessibility

The terminal body is a `role="log"` region whose `aria-live` is set to `off` while printing and restored to `polite` when done, so screen readers are not flooded by per-word DOM changes. Milestone announcements (start, every ~10%, finish) go through the speed indicator, which is `aria-hidden` except for the moment of each announcement.

### Adding a Theme

Add a `[data-theme="name"]` CSS block with all custom properties, and a corresponding `<option>` in the theme `<select>`.

## Testing

Serve locally and test with Playwright MCP:

```
python3 -m http.server 8919
# Then use browser_navigate, browser_snapshot, browser_click, etc.
```

Verify: themes render, scroll animation works with auto-scroll, Stop preserves text, Resume continues from the paused position, Reset clears, GIF exports without console errors, cols/rows/duration/title settings apply correctly and persist across reload. For duration testing, use `browser_evaluate` with a Promise-based timer to measure actual elapsed time vs target. For reduced-motion testing, open a context with `reducedMotion: 'reduce'` via `browser_run_code_unsafe`.
