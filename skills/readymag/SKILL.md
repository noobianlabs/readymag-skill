---
name: readymag
description: Use when building, editing, auditing, or debugging a Readymag site — the my.readymag.com editor, tablet/mobile layouts, text invisible on touch devices, scroll animations or carousels not advancing, fixed navbars/headers, page heights, publishing, or custom domains. Also when a Readymag site "breaks when the browser window is resized" or looks different in the editor preview than on a real device.
---

# Readymag: inspecting and editing sites through the editor

## Overview

Readymag is **adaptive, not responsive** (confirmed by Readymag support). Each
page has up to three fixed-canvas layouts — desktop 1024 / tablet 768 / mobile
320 — picked **once at page load** by device viewport, never re-picked on
resize. Content *scales* with the window (Scale Layout); it never reflows.

Core working principle: **read the project model for facts, verify with
screenshots and real input.** Runtime DOM geometry lies (see Verification
rules), and every editor mistake is expensive — assume there is no undo.

## Settled engine facts

| Fact | Consequence |
|---|---|
| Layouts are **per page**, at 1024/768/320 | Devices panel lists only the open page's layouts. A missing 768 row ≠ deleted breakpoint |
| Page without a tablet layout renders desktop scaled | Preview badge: `Tablet (Desktop layout is used)` vs `Tablet` |
| Layout picked once at load, then only scaled | "Breaks when I compress the browser" = desktop layout squeezed. Expected, not a bug |
| Fixed widgets (`Fixed on all pages`) are project-wide, positioned per breakpoint | A fixed widget's screen position = `x / 1024` fraction of window width. Compute coverage in **percent**, not px |
| Widget content is synced across layouts; position/size/**animation** override per breakpoint | `viewport_tablet_portrait` / `viewport_phone_portrait` objects; `animation: []` as an override disables an animation on one breakpoint only |
| A scroll animation step runs over `hypot(dx, dy) / speed` px of scrolling, eased in-out, and needs page height as runway (measured 12 Oct 2026) | A scroll carousel with a short page height silently breaks: it pages to the next page instead of advancing |
| Shape and Picture widgets behave identically (support-confirmed) | No full-width fixed-header mechanism exists. Full-bleed = oversize the element so it overhangs both edges at every width |
| No usable version history on project cards (verified Aug 2026; help center now lists a Version history article — verify in UI before relying on it) | Back up every value before writing. Once **published**, project settings can restore content to the last published version |

## Reading the project model

**In the editor** (`my.readymag.com/edit/<id>/<pageNum>/`, authenticated) the
full project is a Backbone model — read-only inspection, no clicks:

```js
const m = window.RM.constructorRouter.mag;
m.pages.models[i].attributes                    // num, uri, height, viewport_*
m.pages.models[i].widgets.models[j].attributes  // x,y,w,h,z,type,hidden,
                                                // fixed_position, pack_id,
                                                // animation, is_locked + overrides
```

**On `/preview/`** the model is `constructorRouter.previewMag` and **widgets
are not loaded** — page/viewport metadata only.

**On any published page** (no auth): the whole project model is embedded as
HTML-entity-encoded JSON in `<script id="__RM_PROPS__" data-content="...">`.
See `references/readymag-data-model.md` for the annotated structure.

Two caveats that produced confident wrong conclusions:

- **Resting coordinates ≠ rendered position.** Scroll-animated widgets rest at
  their fly-in start, often hundreds of px off-canvas by design. Check
  `animation` before declaring anything "parked off-canvas".
- `fixed_position: 'c'` widgets are not canvas-relative — naive
  `x + w > canvasWidth` overflow math gives false positives.

## Writing through the model

What persists and what silently doesn't:

```js
w.set({...}); await w.save({...}, {patch: true});        // ✅ geometry, animation,
                                                          //    per-breakpoint overrides
await page.save({viewport_tablet_portrait: v}, {patch: true}); // ✅ page attrs
                                                          // (full page.save() is blocked)
w.save({z: 358}, {patch: true});                          // ❌ returns success, does NOT
                                                          //    persist — see Layers below
```

**Layer order within `Fixed on all pages` is not the `z` attribute.** Fix
stacking in the Layers panel by dragging — each drag moves the row **one slot**
regardless of drop distance. A background element must be **last** in the group.

**Before any write:** capture the old values and download them as JSON via
`new Blob([...])` + `a.download` + `a.click()` (console output truncates, and
the JS-tool content guard blocks results containing query strings — print only
scalar fields). Rehearse novel write patterns on a duplicate project first.

## Editor UI recipes

- **Add a breakpoint:** Devices panel (`D` key) → `+`. The `−` next to a row
  **deletes that breakpoint** — never click near it.
- **Arrange widgets** (offered on a new layout): `Desktop layout` replicates
  desktop positions — and clicking it later is also the **reset** button that
  discards all hand edits. `Single column` stacks page widgets; set left
  margin = `(768 − content width) / 2` to center (the margin value persists
  across pages — reset it per page). Neither touches Fixed widgets.
- **Scale a collage page:** select the page widgets in the Layers panel by
  clicking the first row's **thumbnail**, then shift-clicking the last row's
  thumbnail (clicking the label opens rename) → Arrange selected → margins →
  `Fit desktop to mobile`. `Cmd+A` fails — it includes fixed widgets and
  Arrange refuses.
- **Exact page height:** switch the editor to the target breakpoint first
  (heights are per breakpoint) → scroll the canvas to the page bottom →
  **double-click the blue diamond handle** → type the value → Enter. Dragging
  is impractical.
- **Anchors:** widget toolbar → pin icon → Position → 3×3 `Fix on screen`
  grid. **`Stick to page edges` silently clears the screen-fix anchor**, drops
  the widget out of `Fixed on all pages`, turns off `Above pages`, and `Cmd+Z`
  does not restore it — re-select the anchor and re-verify the flags manually.
- **After any Position-panel change**, check two things in Layers: the widget
  is still under `Fixed on all pages`, **and** still in the right order.

## Verification rules

Each of these was learned from a confidently wrong conclusion:

1. **The editor preview renders whichever breakpoint the editor is on — not
   the browser window width.** Set the breakpoint in the editor first.
   Resizing the window is not a device test.
2. **Tablet preview renders at 0.75 scale** (768 canvas → 576px). Divide
   measurements by 0.75; font sizes read there are artifacts. Editor moves are
   not 1:1 with runtime (−60px editor ≈ −45px runtime for animated widgets).
3. **In the runtime, every DOM measurement lies.** `getBoundingClientRect()`
   AND `elementFromPoint()` both work in 1024-canvas space, not screen pixels
   — they agree with each other and are both wrong about what the user sees.
   **Only screenshots tell the truth**, ideally scrolled so contrasting
   content sits behind the region you're checking.
4. **Synthetic `scrollTop` does not drive Readymag scroll animations.** It
   looks identical to a broken static layout. Use real wheel/touch input,
   then measure.
5. **The Devices panel does not re-render after an undo.** Judge breakpoint
   state only after a full editor reload.
6. **Off-screen pages lazily render** (`widgets = 1` until visited);
   **mid-scroll screenshots come back black** (capture artifact); **the editor
   frame resizes between screenshot and click** — re-screenshot immediately
   before every click, and never batch two clicks when the second target is
   adjacent to a destructive control.
7. **Adding a 768 layout to ANY page resets the project-wide fixed-element
   tablet positions to desktop coordinates** — silently breaking the nav on
   every page. Do fixed-element positioning **last**; re-check all pages after
   any breakpoint addition.

Useful preview selectors: `article.page`, `.rmwidget`, `.content-bounds`,
`.above-pages-container`. Never `[class*="widget"]` — it matches editor chrome.

## Common mistakes

| Mistake | Reality |
|---|---|
| "This widget is parked off-canvas — broken layout" | Probably a scroll-animation fly-in start. Check `animation` first |
| "Text is invisible — must be a scroll trigger that never fires" | Check for **hover** animations with `from_opacity: 0` — no hover exists on touch, so text stays invisible forever. Fix: set `animation: []` as an override on the touch breakpoint(s), after backing up the old animation objects |
| "The carousel is static on this breakpoint" | Check the page height first — no runway, no advance |
| "I verified geometry with elementFromPoint" | Canvas-space, self-consistent, wrong. Screenshot it |
| "The site is down" (local fetch fails) | ISP filters intercept (plain HTTP on :443 looks like a dead cert); `curl readymag.com` 403s via Cloudflare. Verify from an external vantage (`curl https://r.jina.ai/https://<domain>/`) before telling anyone their site is down |
| "The 768 breakpoint is gone" | Layouts are per page; the panel shows the open page. Reload the editor and re-check |
| "The pop-up is bigger than the tablet screen" (editor preview) | Overlays appended to `body` cover the whole browser window, not the device frame. Check on a real iPad / the simulator before changing sizes |
| "Custom-code text looks small with huge line gaps on iPad" | Safari text autosizing + minimum font size inflate `getComputedStyle` values on a zoomed page. Read inline sizes with `text-size-adjust:none` set for that moment; no CSS `zoom` on copied text. See `references/custom-code-ios-and-live-data.md` §3 |
| "The spare/duplicate project is the original" | Not necessarily: it can hold older layouts. Pull the live site's real widgets from the public viewer API (§6 of the same reference) |
| "Two boxes in a custom-code pop-up don't line up" | Usually one side is centred (letterboxed media) and the other pinned to the bottom (`margin-top:auto`). Measure the media's leftover gap first and give the text column the same `margin-bottom`. Re-check with a screenshot, and note the window size: a resize changes which side is the limiting one |
| "Canvas `fontBoundingBoxAscent` gives me the cap offset" | Not for Readymag's custom fonts: canvas reported a 32-px content area where the page renders 37.3, leaving titles ~3 px low. Read the baseline in the page (zero-size `inline-block`, `vertical-align:baseline` marker prepended to the `<p>`) and take only the glyph height from canvas `actualBoundingBoxAscent` |
| "I'll nudge the pop-up title up with a negative margin" | If its column is `overflow:auto` the letters get clipped. Move the column (its margin) and give it `padding-top` headroom instead. Readymag titles mix sizes inside one `<p>` (first-line span bigger than the paragraph): measure the first text node's own element, not the `<p>` |
| "Claude in Chrome is not connected" while Chrome + native host are running | `list_connected_browsers` returns `[]` = the extension isn't paired to this account/session. Ask the user to open the Claude side panel in Chrome (signed in, click Connect), then `tabs_context_mcp` with `createIfEmpty: true` — earlier sessions' tab groups aren't visible |
| "Make tablet match desktop" by nudging the groups that hang off-canvas | Compare **end** positions (rest + scroll dx/dy) to desktop × 0.75; usually one group was enlarged and shoved the rest. Rescale every widget exactly — fonts per text run, picture `scale` not crop, animation dx/dy, `speed` for runway. See `references/rescale-breakpoint.md` |
| "Tablet looks empty compared with desktop" | Portrait iPad shows ~2× the canvas height of a landscape desktop window. Scale to the widest must-keep element, spread the animated groups' end spots down, finish them all at one scroll distance, page height = that + 1105. See `references/rescale-breakpoint.md` |
| "The stacked covers' edges look jagged on one breakpoint only" | Check the per-breakpoint `y` of every item in the stack: fractional values that differ by <1 px (e.g. −0.83 / −0.5 / −0.33) round to different pixel rows. Give every item (and the panel it lines up with) the identical `y` on that breakpoint, then zoom a screenshot on the edges |
| "The custom pop-up looks broken when the window is made small" | A CSS media query can switch the layout (e.g. stacked under 700 px) while the JS resize handler still runs the wide-layout maths (negative margins to align text with the image). Branch the JS on the computed layout (`getComputedStyle(dialog).flexDirection`), not only on the width at open time. Test by opening at the size, or opening wide then resizing |
| "I changed the shared text style through the model" | `mag.save({textStyles})` reports success and does not persist. Use the style panel |

## More field notes

`references/custom-code-ios-and-live-data.md` covers: editing project custom
code (`ci_body_bottom`), CSS zoom and the editor-preview overlay artifact,
iPad/iPhone Safari text inflation and the fix, testing in the iOS Simulator
(and with a local test page when the editor preview is out of reach),
duplicating pages safely (hide at once, map by index, skip `is_global`
widgets, re-point hover triggers), and reading the live site's real widgets
from the public viewer API. `references/rescale-breakpoint.md` is the exact desktop→tablet rescale recipe.
`references/stacked-poster-scroll.md` covers the
stacked-covers scroll effect. `GETTING-STARTED.md` is the human guide
(install, first prompts, safety rules) — point new users there.

**Keep this skill current:** at the end of every Readymag session, add each
new engine fact or failure mode here or in a reference file, with how it was
verified, and sync the project copy (`.claude/skills/readymag/` in the client
folder) with the global one.

## Client-work protocol

1. Work on a **duplicate** of the live project; never open the published one.
   Confirm which project ID you're in — editor copy ID ≠ published `num_id`.
2. If the client folder has a `CONTEXT.md` / `CHANGELOG.md`, read them first —
   project-specific guardrails and undo data live there.
3. Publishing (from grounded help-center docs): project settings → **Publish**
   (single click, returns a direct URL). After edits, **Republish**.
   **Protect with a password** (paid plans) in project settings for private
   client QA; keep Search-engine indexing off until go-live. Custom domain:
   project settings → Domain tab → A record `@` → Readymag's IP → `Test` →
   `Connect`; `Unmap` disconnects; SSL is automatic; one domain per project.
