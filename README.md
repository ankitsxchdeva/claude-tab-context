# claude-tab-context

**Label every terminal tab with what its Claude Code session is doing.**

Running five Claude Code sessions across five tabs and squinting to remember
which is which? This sets each tab's title to the session's **project** and
**first prompt**, with a status glyph — so you can triage the whole tab bar at
a glance.

```
▸ dots · fix backspace in ghostty           ← working
✓ api  · add rate limiting to /v1/search    ← done, waiting for you
● web  · migrate to tailwind v4             ← needs your input
```

One dependency-light shell script (just `jq`), wired through Claude Code hooks.
Works in any terminal that honors OSC 2 titles — Ghostty, iTerm2, kitty,
WezTerm, Terminal.app, VS Code's integrated terminal.

## Install

```
/plugin marketplace add ankitsxchdeva/claude-tab-context
/plugin install claude-tab-context@claude-tab-context
```

Open a new session and your tabs start labeling themselves. That's it.

## What the title shows

`<status> <project> · <prompt>`

- **status** — `▸` working · `✓` done (waiting for your next message) · `●`
  needs input (a permission prompt or other notification)
- **project** — the git repo name (or the working directory), leading dot
  stripped (`.dots` → `dots`)
- **prompt** — the session's **first** prompt, so quick follow-ups
  ("commit this", "now fix the test") don't relabel the thread

## Configuration

All optional, set as environment variables:

| Variable | Default | Effect |
|---|---|---|
| `CTC_SEP` | `" · "` | separator between project and prompt |
| `CTC_MAXLEN` | `40` | max prompt length |
| `CTC_PROJ_MAXLEN` | `16` | max project length |
| `CTC_WORKING` / `CTC_DONE` / `CTC_WAITING` | `▸` / `✓` / `●` | status glyphs |
| `CTC_LATEST` | unset | set to `1` to label with the latest prompt instead of the first |
| `CTC_TTY` | unset | force a target tty device (skip auto-detect) |

## Terminal notes

- **Ghostty:** if you've set a fixed `title = "..."` in your config, Ghostty
  **ignores all title escape sequences** — comment it out or this can't work.
  (A one-time gotcha, not specific to this plugin.)
- **tmux / screen:** the title is set on the pane's tty. To bubble it up to the
  outer terminal tab, add `set -g set-titles on` so tmux forwards the pane
  title.
- Any terminal honoring **OSC 2** works; there's no terminal-specific code.

## How it works (and a caveat)

Claude Code runs hooks **without a controlling terminal**, so the usual
`/dev/tty` write fails with `ENXIO` ("Device not configured"). This script
instead finds the session's real tty *device* (e.g. `/dev/ttys003`,
`/dev/pts/3`) by walking up the process tree, and writes the `OSC 2` sequence
there.

Newer Claude Code versions expose a hook-output `terminalSequence` field that
emits titles through Claude's own write path (race-free, tmux- and
Windows-safe). Where it's available that's a cleaner mechanism; this script's
tty approach is the dependency-free fallback that works today everywhere there
is a real tty.

## Standalone (no plugin)

Prefer to wire it into your own dotfiles? Copy `scripts/claude-tab-context` to
`~/.claude/hooks/` (make it executable with `chmod +x`), then merge the hooks
from [`examples/settings.snippet.json`](examples/settings.snippet.json) into
your `~/.claude/settings.json`.

## Prior art

Several projects label Claude tabs — [cc-tab-titles](https://github.com/STRML/cc-tab-titles),
[franzvill/claude-code-tab-title](https://github.com/franzvill/claude-code-tab-title),
[JasperSui/claude-code-iterm2-tab-status](https://github.com/JasperSui/claude-code-iterm2-tab-status).
This one's niche: **minimal, no framework, and the label carries project + real
prompt context** rather than just busy/idle or an AI-generated summary.

## License

MIT © Ankit Sachdeva
