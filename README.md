# ableton-audio-output

**Switch Ableton Live's audio output device from the command line, on macOS — in the
running app, without quitting or restarting it.**

```console
$ live-output --list
No Device
Use System Interface
Background Music (2 In, 2 Out)
MacBook Pro Speakers (0 In, 2 Out)
✔ P-Series (2 In, 2 Out)

$ live-output MacBook
OK: MacBook Pro Speakers (0 In, 2 Out)
```

`live-output` with no argument prints what Live is playing out of right now; with a
pattern, it switches to the first output whose name contains it.

## What it's for

Changing Live's audio output is a manual trip through Settings → Audio → Audio Output
Device, every time: there is no shortcut to bind, no MIDI mapping, no menu item. That is
an annoyance at a desk and a real problem on stage, where a laptop that woke up on the
wrong device is silent at the first note and nobody opens a settings dialog in front of
an audience.

## Install

```bash
git clone https://github.com/Beennnn/ableton-audio-output.git
cd ableton-live-output && ./install.sh      # symlinks into ~/.local/bin
```

Then allow whatever **calls** the script in **System Settings → Privacy & Security →
Accessibility** — that one grant is enough, and it is granted per calling process: a
terminal you authorised does nothing for a background service running the same script.
When it is missing you get exit code 4 and a line saying so, rather than an AppleScript
error number.

## Exit codes

| code | meaning |
|---|---|
| 0 | done, or state printed |
| 1 | bad usage, or Live is not running |
| 2 | no output matches the pattern |
| 3 | the change did not take |
| 4 | the caller lacks Accessibility permission |

Code 4 covers the same refusal seen from three angles, because the repair is identical in
all three: `-25211` (reading the interface denied), `1002` (sending input denied) and
`-1743` (sending Apple events denied). Machine-readable on purpose: a caller should never
have to parse a sentence.

## Tested on

Live 12.4.2, macOS 26 (Tahoe), Apple Silicon. Live 11 and earlier expose a different
accessibility tree and are **not** supported.

## How it works, and why this way

It drives Live's settings window through the macOS accessibility layer, then reads the
value back to confirm the change took before reporting success. Every cleaner route —
AppleScript, `Preferences.cfg`, Live's Python API, Max for Live — is a dead end, and the
page is found by structure rather than by any translated label, so it survives a change
of interface language. The full reasoning, why one permission is now enough where two
were being asked for, and the three traps already walked around are in
**[docs/design-notes.md](docs/design-notes.md)**.

## Licence

MIT.
