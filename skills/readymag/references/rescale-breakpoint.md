# Rescaling a desktop page onto the tablet (or phone) breakpoint exactly

Use when someone asks for a breakpoint to "match desktop, just smaller". Verified 12 Oct 2026 on a
57-widget collage page (a Home page on a working copy): written through the model, persisted after reload,
checked in the tablet preview with real wheel input.

## Why the existing layout drifts
Earlier hand passes tend to enlarge one group for legibility (e.g. keep 9-px copy instead of 6 px).
The enlarged group then pushes its neighbours sideways, and their scroll animations end off-canvas.
Spot it by computing each widget's **end** position (resting x/y + the sum of scroll `dx`/`dy`) on both
breakpoints and comparing tablet end to `desktop end × k`.

## The recipe (k = 0.75 for 1024 → 768; add a y offset if the tablet header is taller)
Per visible, non-fixed, non-global widget, write `viewport_tablet_portrait = {...old, ...new}`:

| Field | New tablet value |
|---|---|
| x, y, w, h | desktop · k (y · k + header offset) |
| angle | desktop angle |
| picture `scale` | desktop scale · k. `crop*` / `originalCrop*` are **source-image px**: copy the desktop values unchanged |
| text `styles` | rebuild every block's `inlineStyles`: per character take the desktop inline FONT_SIZE, or the block's text-style size (`mag.get('textStyles').project[].cssProperties.fontSize`, matched by `blocksMeta[].data.textStyle`) when there is none; write FONT_SIZE = size · k on every run. Same for LETTER_SPACING. Keep other inline keys (DECORATION, FONT_WEIGHT, TRANSFORM …) |
| text `blocksMeta` | copy desktop, `lineHeight` (or the style's) · k, numeric paddings · k |
| scroll `animation` | copy desktop, `dx`/`dy` · k. If `|dy|·k` exceeds the page's runway, raise `speed` (`max(1, |dy|/runway)`) instead of capping dy — the end position stays exact |

`inlineStyles` format: `{styles:[{offset,length,styles:["keyIdx;valueIdx",…]}], keys:[…], values:[…]}`.
Fractional font sizes (6.75, 9.75) are accepted and render.

Skip: `background` widgets, fixed/global widgets (nav, logo), widgets hidden on desktop.
Leave `z` alone (layer order on a breakpoint is the Order popover's, not `z`).

## Trade-off to tell the client
An exact ×0.75 makes desktop 8-px copy 6 px on the 768 canvas (≈ 6.4 px on an iPad in portrait). That is
the price of "nothing cut off, same composition". Check on a real iPad: Safari's minimum font size
can enlarge it and reflow the text box.

## Undo
Download the whole page model first (`{page: p.attributes, widgets: p.widgets.models.map(w => w.attributes)}`)
and restore each widget's `viewport_tablet_portrait` from it.

## Filling a portrait tablet screen (when ×0.75 looks empty)
A landscape desktop composition scaled to 768 fills only ~40% of a portrait iPad (editor tablet preview shows
768 × ~927 canvas px; iPad Air ≈ 768 × 1105). What worked (12 Oct 2026, Home page):
1. Scale to the widest element that must not be cut (here the folder: 905 → 760, ×0.84), not to 1024.
2. Keep resting positions at that scale; spread the **end** positions of the animated groups down the screen
   (move each group rigidly; new top = target + (top − topmost)·k) and nudge any group back inside the edges.
3. Make every group finish at one scroll distance E (≈ when the opening element has scrolled under the nav):
   `speed = hypot(dx, dy) / E`.
4. Give a static element that sat below the fold on desktop a gentle tablet-only scroll drift so it fills the
   space under the opening view at load and lands under the cards at the end.
5. Tablet page height = E + 1105 (largest common iPad viewport), or the animations stop short on a real iPad.

## Scroll-animation timing (measured 12 Oct 2026 in the tablet preview)
- One step runs over **hypot(dx, dy) / speed** canvas px of scrolling — the diagonal length, not dy alone
  (a board with dx −316, dy 265 at speed 0.8 took ~515 px, not 330).
- Progress is eased (ease-in-out / smoothstep), so mid-scroll positions are not linear.
- Widgets already on screen at load start at scroll 0.
- Read progress in the preview from the rotation in the widget's parent transform
  (`getComputedStyle(el.parentElement).transform`, `atan2(b, a)` ÷ the step's `rotate`); wheel ticks are
  133.3 CSS px = 177.8 canvas px at the 0.75 preview scale.

## When the client wants the desktop *arrangement*, not a full screen
12 Oct 2026: after the spread-out version, the client sent the desktop end state and asked to match it. Clients
judge by the arrangement (what overlaps what, what lines up), so keep the relative end positions exact (no
vertical spread) and pick the largest uniform scale whose end composition fits 768 after small edge nudges
(×0.80 for a composition 1000 desktop px wide). Show them early that a landscape arrangement leaves the lower
part of a portrait iPad empty; offer the spread only as an alternative.
