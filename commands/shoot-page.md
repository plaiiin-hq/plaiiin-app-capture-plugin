---
description: Render a web page to a PNG — no Screen Recording needed
---

Photograph the page: **$ARGUMENTS**

Use `page_shot`, which renders offscreen and needs no permission at all. Sensible defaults:
1440×900, scale 2. Reach for `appearance: "both"` when they want light and dark, `fullPage` for
the whole scroll height, `waitFor` when the part they care about arrives late.

Write it where they asked, or `~/Desktop/`, then show it.
