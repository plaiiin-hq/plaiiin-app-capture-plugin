---
name: app-capture
description: Use when you need to SEE the Mac you are working on — a screenshot of an app's window, a region of the screen, or a rendered web page — or when a capture comes back looking wrong (a picture of the wallpaper, a window with something across it, a postage stamp instead of a window). Covers the four tools, staging a window before the shot (hide the rest, centre, fit, exact size, all restored afterwards), poster backdrops drawn rather than photographed, and the refusals, which are the most useful thing this tool says.
---

# Seeing the Mac you are working on

plaiiin App Capture photographs a window, a rectangle or a web page when asked, over MCP. It runs
on the person's own Mac; the server is a Unix socket held by a login agent, so it answers what it
can do while the app itself is closed and starts the app only when a capture actually arrives.

**Nothing is captured unless you ask.** There is no watching, no polling, no screen sharing.

## The four tools

| tool | for | needs Screen Recording |
|---|---|---|
| `probe` | whether capturing is possible at all, and under which identity | no |
| `list_windows` | every on-screen window with id, owner, title, frame | no |
| `shot` | one PNG of a window (by `owner`/`title`) or an explicit `rect` | **yes** |
| `page_shot` | one PNG of a URL, rendered offscreen in a WKWebView | no |

`page_shot` needs no grant at all and cannot catch another app's window, so prefer it whenever the
thing you want to see is a page rather than an app.

## Read the refusal, it is the answer

This tool refuses rather than returning a picture that looks right and is not. macOS, left alone,
does the opposite: without the Screen Recording grant a capture succeeds and returns a correctly
sized photograph of the **wallpaper**, with no window in it, and every log reads green.

| refusal | what to do |
|---|---|
| "no Screen Recording grant" | tell the person; only they can grant it (System Settings ▸ Privacy & Security ▸ Screen & System Audio Recording). Nothing works around it |
| "no window matched … but N windows of that name exist off this desktop" | it is on another desktop, in a full-screen space, or minimised. Retry with `"isolate": true`, which brings it forward first |
| "parked off-stage in Stage Manager, so its window is listed at 139×144" | same fix: `"isolate": true`. Never shoot it as-is — that rect is the strip, not the window |
| "did you mean 'Safari'?" | the app name was close but wrong; `owner` is the app's display name, not its bundle id |
| "window not moved: … Accessibility" | `centre`/`fit`/`size` need that grant; the shot still happened, unmoved |

A `shot` reply carries `window` (the frame it found), and `notes` when something it was asked to do
could not be done. Read `notes` — the picture may be fine and the staging may not have been.

## Staging, when the window is not photogenic

All opt-in, all restored afterwards — including when the shot fails.

```json
{"name": "shot", "arguments": {
  "owner": "Workflows", "output": "~/Desktop/workflows.png",
  "isolate": true, "center": true, "size": {"width": 1440, "height": 900}
}}
```

| argument | does |
|---|---|
| `isolate` | hides the other apps, brings this one forward (also the fix for Stage Manager and other desktops) |
| `center` · `fit` · `size` | centre it, pull one hanging off a screen edge back on, or give it an exact frame. Need Accessibility |
| `margin` | points of space around the window |
| `backdrop` | fill that margin with something other than the desktop — see below |

The window is photographed **as itself**, so anything lying on top of it is not in the picture even
without `isolate`.

## Poster shots

`margin` alone photographs the desktop around the window — wallpaper, other windows, the Dock.
`backdrop` draws it instead, so a set of shots looks the same on every Mac:

| value | is |
|---|---|
| `tea-garden`, `tea-leaf`, `aerial`, `tahoe` | pictures shipped with the app, aspect-filled |
| `spotlight`, `spotlight:#0A84FF`, `spotlight:graphite:halo` | a pool of light in any tint; styles `pool`, `beam`, `halo`, `flat` |
| `graphite`, `black`, `white`, `paper`, `system`, `#RRGGBB` | a flat colour |

```json
{"name": "shot", "arguments": {
  "owner": "Safari", "output": "~/Desktop/poster.png",
  "isolate": true, "margin": 150, "backdrop": "spotlight:#0A84FF:pool"
}}
```

## Pages

```json
{"name": "page_shot", "arguments": {
  "url": "https://example.com", "out": "~/Desktop/page.png",
  "width": 1440, "height": 900, "appearance": "both", "waitFor": "#main"
}}
```

`appearance: "both"` writes the light PNG to `out` and the dark one to its `-dark` sibling from a
single load. `waitFor` waits for an element to exist **and have a size**, so an element revealed by
a script counts as arrived only when it is actually shown. `fullPage` captures the whole scroll
height; `clip` crops to one element.

## Where files go

Paths are the person's own: `~` means their home, not a container. Write where they asked, and
prefer somewhere they will look — the Desktop, or the repo folder the work is in.

## When nothing answers

The socket is `~/Library/Group Containers/LA8GX8Y3R9.group.plaiiin/run/appcapture-mcp.sock`. If the
MCP server fails to connect, the app is not installed: it is a free download from
**https://plaiiin.com/app-capture** (signed and notarized, not on the App Store — a sandboxed app
cannot hold the Accessibility grant that staging needs). Tell the person that rather than guessing;
there is no fallback that produces a real screenshot.
