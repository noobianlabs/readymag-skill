# Stacked poster scroll (horizontal pile-up on scroll)

Source: Readymag's own tutorial, "how to create the horizontal stacked poster scroll"
(youtube.com/watch?v=Z07I1FDPED0, 5:50), inspired by sainthelenastudio.com/posters/.
Values below are what the presenter typed on screen, not guesses.

## What the effect is

Posters start spread out in a straight horizontal line, most of them off-canvas to the right.
As the visitor scrolls, each one slides left and lands **one column short of the one before it**,
so they shuffle into a neat overlapping pile — only a sliver of each earlier poster showing.
It is the ordinary pinned horizontal scroll with **staggered end positions**; that is the only
difference.

## Build, step by step

1. **Pictures, duplicated.** Place one Picture widget, then `Cmd+D` it for each poster and swap the
   image inside. Duplicating (rather than adding new widgets) keeps every poster the same size.
2. **Lay them in a line.** Space them evenly left→right, touching or with padding — a design choice.
   Zoom the artboard out (or scroll sideways) to place the ones off-canvas.
3. **Add a grid to count against.** Grid & Layout → columns **18**, gutter **1**. The grid is only a
   guide: each poster's end position is one column further along than the previous poster's, and the
   columns keep that honest. Tip from the video: park a temporary Shape widget on the column you are
   currently using and move it along as you go, so two posters never land on the same column.
4. **Animate every poster except the first.** Select poster → Animation → **Scroll** → **Move**, then
   drag the poster to where it should finish. Panel values seen on screen:

   | Setting | Value |
   |---|---|
   | Type | Scroll |
   | Step | 1 |
   | Start Point | **Bottom** |
   | Speed | **1x** |
   | Easing | Ease in and out |
   | Move | dx **−286**, dy **0** |
   | Bézier | off |

   dx is negative (leftward) and dy is exactly 0. Speed can be varied per poster to make some
   overtake others. The **first poster gets no animation** — it is the base of the pile.
5. **Pin them, or they will also travel vertically.** Select all the poster widgets → Position →
   pin to the **centre of the screen**. Without this they scroll up with the page as well as
   sideways, which reads as broken. (This is the `fixed_position: 'c'` anchor in the model.)
6. **Give the page runway.** A horizontal scroll animation is driven by vertical scroll distance, so
   the page must be tall. In the video the page height is set to **5,000**. Too short and the
   animation is cut off mid-flight and the site jumps to the next page.

## Options mentioned

- Link each poster out to an article or post; enable **draggable** so visitors can pull them apart;
  enable **lightbox** for full-screen viewing; add **scale on hover** as a second animation.
- Mix in a few Video widgets among the pictures for motion-graphic posters.
- With pages stacked vertically, put ordinary content above and below the stack by adding pages.

## Mobile / tablet

The effect works, but **redo the design and the animation numbers per breakpoint** — a dx tuned for
1024 will not read on a 320 canvas, and page height is per breakpoint too. See the main SKILL.md:
layouts are picked once at load and only scaled, and animation is one of the properties that can be
overridden per breakpoint (`animation: []` disables it on one device).

## Gotchas that bite

- Page height is per breakpoint. A tablet layout inheriting a short height silently truncates the
  strip (this exact bug appeared on a client's Spotify carousel: tablet height 926 against a desktop
  3272 — only 3 of 13 covers were reachable).
- Model coordinates are the animation's **resting** position, i.e. where a poster starts, often far
  off-canvas. Do not read them as "broken layout".
- Verify with real wheel/trackpad scrolling in preview; synthetic `scrollTop` does not drive
  Readymag scroll animations.
