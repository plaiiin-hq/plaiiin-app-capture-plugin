---
description: Photograph an app's window on this Mac and show it
---

Take a screenshot of the window the user named: **$ARGUMENTS**

1. `list_windows` first when the name is uncertain, and pick by what is actually on screen.
2. `shot` with `owner` (and `title` when the app has several windows), writing to the path they
   asked for — or `~/Desktop/<app>-<something>.png` when they did not say.
3. Add `"isolate": true` if the refusal says the window is off this desktop or off-stage.
4. Show them the picture, and say where it was written.

Read the refusal rather than retrying blindly: it names what to do.
