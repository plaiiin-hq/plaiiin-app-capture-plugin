---
name: app-capture
description: Use when you need to SEE the Mac you are working on — a screenshot of an app's window, a region of the screen, or a rendered web page — or when a capture comes back looking wrong (a picture of the wallpaper, a window with something across it, a postage stamp instead of a window), or when the capture server does not answer at all and the app has to be installed or updated. Covers the four tools, staging a window before the shot (hide the rest, centre, fit, exact size, all restored afterwards), poster backdrops staged behind the window so its shadow and glass are real, where to download the app and how it keeps itself current, and the refusals, which are the most useful thing this tool says.
---

# Seeing the Mac you are working on

plaiiin App Capture photographs a window, a rectangle or a web page when asked, over MCP. It runs
on the person's own Mac; the server is a Unix socket held by a login agent, so it answers what it
can do while the app itself is closed and starts the app only when a capture actually arrives.

**Nothing is captured unless you ask.** There is no watching, no polling, no screen sharing.

## The four tools

| tool | for | needs Screen Recording |
|---|---|---|
| `probe` | whether capturing is possible, under which identity, and **which version is installed** | no |
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
| `backdrop` | put a surface behind the window instead of the desktop — see below. Needs `isolate` |

Without a `backdrop` the window is photographed **as itself**, so anything lying on top of it is
not in the picture even without `isolate`.

## Poster shots

`margin` alone photographs the desktop around the window — wallpaper, other windows, the Dock.
`backdrop` puts a surface there instead, so a set of shots looks the same on every Mac:

| value | is |
|---|---|
| `tea-garden`, `tea-leaf`, `aerial`, `tahoe` | pictures shipped with the app, aspect-filled |
| `spotlight`, `spotlight:#0A84FF`, `spotlight:graphite:halo` | a pool of light in any tint; styles `pool`, `beam`, `halo`, `flat` |
| `graphite`, `black`, `white`, `paper`, `system`, `#RRGGBB` | a flat colour |

```json
{"name": "shot", "arguments": {
  "owner": "Safari", "output": "~/Desktop/poster.png",
  "isolate": true, "margin": 150, "backdrop": "spotlight:#12161B:pool:0.85:0.45"
}}
```

`backdrop` requires `isolate: true`, and the reason is worth knowing: the surface is placed on the
screen BEHIND the window and the region is photographed, so the window's own shadow falls on it
and its vibrancy samples it. A border drawn on afterwards has no shadow and its glass still shows
the desktop that was really there.

A spotlight takes `spotlight:<tint>:<style>:<size>:<strength>` — tint is the colour of the dark,
strength is contrast (how far the pool lifts toward white), size is how tight the pool is. Useful
starting points: `spotlight:#12161B:pool:0.85:0.45` (neutral studio), `…:0.9:0.22` (soft, low
contrast), `#0A0D11:pool:0.7:0.75` (near-black, dramatic), `#191310` warm, `#0D1520` cool.

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

## Installing it, when nothing answers

The socket is `~/Library/Group Containers/LA8GX8Y3R9.group.plaiiin/run/appcapture-mcp.sock`. If the
MCP server fails to connect, the app is not installed. There is no fallback that produces a real
screenshot — say so and give the person these steps rather than guessing:

1. Download **https://plaiiin.com/app-capture** — a free, Developer-ID signed and notarized disk
   image, so it opens with no Gatekeeper warning. It is deliberately **not** on the Mac App Store:
   Apple gives a sandboxed app no Accessibility grant at all, and placing a window before a shot
   needs one.
2. Open the image, drag **plaiiin App Capture** to Applications, and open it once. That first
   launch registers the login agent that answers this socket.
3. Permissions are asked for only when first needed, and only the person can grant them:
   **Screen Recording** at the first `shot` (web pages need none), **Accessibility** at the first
   `center`/`fit`/`size`/`backdrop`. After granting either, the app must be quit and reopened —
   macOS only hands a grant to a fresh process.

## Keeping it current

`probe` reports `version`, `build` and the `updates` feed, so you can say exactly which copy is
installed rather than guessing at symptoms.

Updates install themselves: the app checks daily and applies what it finds without asking. A
person can force it from the app's menu — **plaiiin App Capture ▸ Check for Updates…** — and there
is no tool for an agent to trigger it, on purpose: an app that rewrites itself because something
asked it to is not one you would leave running.

If a capability described here is missing and `probe` shows an old `build`, that is the answer:
the copy is behind, and one Check for Updates… fixes it.
