# plaiiin App Capture — Claude Code plugin

Ask Claude to look at your Mac. This plugin wires [plaiiin App
Capture](https://plaiiin.com/app-capture) into Claude Code, so an assistant can photograph a
window, a region or a web page while you talk about it.

```
/plugin marketplace add plaiiin-hq/plaiiin-app-capture-plugin
/plugin install plaiiin-app-capture
```

Then: *"shoot the Workflows window"*, *"poster shot of Safari on the tea-garden backdrop"*,
*"render plaiiin.com in light and dark"*.

## What you need

The app itself — a free, signed and notarized download from
**[plaiiin.com/app-capture](https://plaiiin.com/app-capture)**. It is not on the Mac App Store,
because a sandboxed app cannot hold the Accessibility permission that placing a window needs.

**Nothing runs while you are not using it.** A small login agent holds the socket and answers what
the tool can do; the app itself starts only when a capture actually arrives, and goes away again.

## What it will not do

It refuses rather than returning a picture that looks right and is not. Without the Screen
Recording grant macOS hands back a correctly sized photograph of your wallpaper with no window in
it — this tool checks first and says so. A window on another desktop, a name that matches nothing,
a folder it may not write to: each is an error that says what to do about it.

Nothing leaves your Mac. Captures are written where you asked and stay there; the only network
traffic is a page you explicitly ask it to photograph.

## What it can do

| | |
|---|---|
| `shot` | a window by app name and title, or an explicit rect — a real screen capture |
| `page_shot` | a URL rendered offscreen; needs no permission, light and dark in one call |
| `list_windows` | everything on screen, so you can name a window instead of guessing |
| `probe` | whether capturing is possible, and under which identity |

Staging, all optional and all restored afterwards: hide the other apps, bring the window forward,
centre it, pull one that hangs off the screen edge back on, or give it an exact size. Poster
backdrops are drawn rather than photographed — a colour, a generated pool of light in any tint, or
one of the pictures the app ships with — so your wallpaper is never touched.

Apache-2.0.
