# Terminal Scroller

A single-file web tool that emulates a terminal scrolling through long output. Text prints word by word, starting at a readable pace and accelerating exponentially to emphasize output volume.

No dependencies. No build step. Just open `index.html` in a browser.

## Usage

1. Open `index.html` in any modern browser
2. Paste or type your text in the textarea
3. Click **Start** to begin the scrolling animation
4. Click **Stop** to pause (preserves output), then **Resume** to continue from where it left off, or **Reset** to clear. Changing the speed mode or duration while paused restarts the animation with the new pacing.
5. If your OS is set to reduce motion, output appears instantly instead of animating

The output starts at a readable word-by-word pace and progressively accelerates. For very long text, the final words blur past at maximum speed — the contrast emphasizes just how much output there is.

## Speed Modes

A radio toggle selects between two output modes:

- **Accelerating** (default) — starts at a readable pace and exponentially accelerates. Emphasizes output volume by contrasting the readable start with a blurred finish.
- **Constant** — uniform speed throughout. Every word prints at the same interval.

Both modes respect the Duration setting.

## Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Cols** | Terminal width in character columns | 80 |
| **Rows** | Terminal height in text rows | 20 |
| **Duration (s)** | Target animation duration; leave empty for auto-pacing | auto |
| **Title** | Text shown in the terminal titlebar | ~/output |

When a duration is set, the pacing curve scales to fit. In accelerating mode, the start always remains readable (~60ms/word) even with short durations and long text — the acceleration curve steepens to compensate. In constant mode, the delay is simply target time divided by word count. Timing is scheduled against elapsed wall-clock time, so throttled or delayed timers self-correct and the target duration stays accurate.

All settings (speed mode, cols, rows, duration, title) are saved to localStorage and restored on the next visit. Numeric values are clamped to their valid ranges.

## Themes

20 built-in terminal themes, selectable via dropdown. Choice is saved to localStorage.

Claude Code, Green Phosphor, Amber CRT, Matrix, Homebrew, Monokai, Dracula, Solarized Dark, Solarized Light, Nord, Gruvbox, One Dark, Tokyo Night, Catppuccin Mocha, Rose Pine, Cobalt2, Material, Ubuntu, PowerShell, Minimal

## GIF Export

Click **Export GIF** to render the scrolling animation as an animated GIF. The export uses the current theme, terminal dimensions, and title, and renders text with the same font stack as the page. Frame sampling is adaptive — early words get individual frames for word-by-word fidelity, while the fast tail is batched into fewer frames. The maximum frame count scales down for large canvases to bound memory use. Requires an internet connection on first use (loads gif.js from CDN; the pinned release is verified against SHA-384 digests before it is executed).

## Adding a Theme

Add a `[data-theme="name"]` CSS block in `index.html` with these custom properties:

```
--bg, --text, --accent, --term-bg, --term-border, --titlebar,
--input-bg, --input-border, --btn-bg, --btn-text, --btn-hover, --glow
```

Then add a matching `<option>` to the theme `<select>` element.
