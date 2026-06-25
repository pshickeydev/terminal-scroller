# Terminal Scroller

A single-file web tool that emulates a terminal scrolling through long output. Text prints word by word, starting at a readable pace and accelerating exponentially to emphasize output volume.

No dependencies. No build step. Just open `index.html` in a browser.

## Usage

1. Open `index.html` in any modern browser
2. Paste or type your text in the textarea
3. Click **Start** to begin the scrolling animation
4. Click **Stop** to pause (preserves output) or **Reset** to clear

## Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Cols** | Terminal width in character columns | 80 |
| **Rows** | Terminal height in text rows | 24 |
| **Duration (s)** | Target animation duration; leave empty for auto-pacing | auto |
| **Title** | Text shown in the terminal titlebar | ~/output |

When a duration is set, the pacing curve scales to fit — the start stays readable (~60ms/word) and the acceleration steepens to hit the target time. For very long text with short durations, words are batched to overcome browser timing limits.

## Themes

20 built-in terminal themes, selectable via dropdown. Choice is saved to localStorage.

Claude Code, Green Phosphor, Amber CRT, Matrix, Homebrew, Monokai, Dracula, Solarized Dark, Solarized Light, Nord, Gruvbox, One Dark, Tokyo Night, Catppuccin Mocha, Rose Pine, Cobalt2, Material, Ubuntu, PowerShell, Minimal

## GIF Export

Click **Export GIF** to render the scrolling animation as an animated GIF. The export uses the current theme, terminal dimensions, and title. Frame timing matches the pacing curve. Requires an internet connection on first use (loads gif.js from CDN).

## Adding a Theme

Add a `[data-theme="name"]` CSS block in `index.html` with these custom properties:

```
--bg, --text, --accent, --term-bg, --term-border, --titlebar,
--input-bg, --input-border, --btn-bg, --btn-text, --btn-hover, --glow
```

Then add a matching `<option>` to the theme `<select>` element.
