# Why it drives the interface, and how

## There is no cleaner route

Worth knowing before you reach for something tidier: there is nothing tidier.

| Route | Why it doesn't work |
|---|---|
| AppleScript | Live ships **no scripting dictionary** — no `.sdef` in the bundle |
| `Preferences.cfg` | Binary, UTF-16 keys, and **the device name is not in it in readable form**. Live also rewrites the file when it quits, so editing it under a running Live changes nothing |
| Live's Python API | Control-surface scripts reach tracks, clips and devices — **not the audio hardware** |
| Max for Live | Same limit: it lives inside the audio engine, it does not choose it |

What is left is the accessibility layer, and Live 12 exposes it properly: the settings
window publishes its pop-up buttons and the device list is enumerable. The script drives
that, then **reads the value back** to confirm the change actually took before reporting
success.

## Accessibility alone is enough

The script never needs *Input Monitoring* or a second "send keystrokes" grant. It used
to: `⌘,` opened the settings window and `esc` dismissed a drop-down, and both are
keystrokes, which macOS gates separately from reading the interface. A process holding
only the read grant therefore failed every time while everything else about it worked.
Both are now done through the accessibility tree — a click on the menu item, and
`AXCancel` on the open menu — with the keystroke kept only as a fallback.

## It does not read a single translated string

Window and tab titles follow Live's UI language, so relying on them would break the day
the interface is in another language. The audio page is found by **structure** — it is
the only settings page holding three pop-up buttons (driver type, input device, output
device) — and the output is the third from the top.

The device names themselves come from CoreAudio, and are matched as a substring,
case-insensitively (AppleScript's `contains`).

## Three traps this already walks around

**One permission where two were being asked for.** See the section above: opening
settings and closing a drop-down went through keystrokes, which macOS grants separately
from reading the interface. The menu item is clicked at **position 3** rather than by its
label (`About Live | — | Settings… | — | Services | …`), which keeps the rule this
project holds to: never depend on a translated string.

**Waiting for a delay instead of for the thing.** Opening the settings window takes up to
three seconds; a fixed pause is wrong both ways — too short and the first call fails, too
long and every call drags. The script polls until the element exists. The symptom before
that fix was distinctive: the first call failed, the next two worked, because they found
the window already open.

**A here-document inside `$( )`.** Bash 3.2 — the `/bin/bash` that macOS ships, and the
one a launchd service gets — rejects it: *unexpected EOF while looking for matching `'`*.
Homebrew's bash 5 accepts it. So the script ran fine from a terminal and failed only when
called by a service. It now writes through a temporary file instead.

## The installer symlinks, it does not copy

`install.sh` puts a **link** in `~/.local/bin`, not a copy: a copy starts diverging from
the repository at the first fix, and the repository has to stay the source.
