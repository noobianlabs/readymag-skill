# Custom code, iPad/iPhone Safari, duplicates and live-site data

Field notes from a real client project (Oct 2026). Each item was learned
from a real failure; the "verified" line says how.

## 1. Project custom code

- Settings → Code → "before </body>" is the mag attribute `ci_body_bottom`
  (HTML string with `<style>` + `<script>`). Read it with
  `RM.constructorRouter.mag.get('ci_body_bottom')`; write it with
  `m.set({ci_body_bottom: s}); await m.save({ci_body_bottom: s}, {patch: true})`.
  It persists (verified by reload). Back it up first (Blob download + a
  localStorage key in the editor tab).
- Syntax-check the `<script>` part with `node --check` before saving: a parse
  error silently kills every feature in the block.
- Shared **text styles** are different: `mag.save({textStyles}, {patch:true})`
  reports success but does NOT persist. Change styles in the editor's style
  panel (⌘-click text → Aa → T → ✎ → Save).
- Code that targets widgets by `data-id` only works on the page whose widgets
  it names. A duplicated page gets new ids, so the code does nothing there.

## 2. Page scale and CSS zoom

- The page scale is CSS `zoom` on `.page-content-container` (product of the
  ancestors' computed `zoom`). Custom overlays appended to `body` sit outside
  it, so size them as `canvas px × zoom`.
- In the editor preview, an overlay appended to `body` covers the WHOLE browser
  window, not the simulated device frame. A tablet-preview pop-up therefore
  looks bigger than the "screen". It is a preview artifact; on a real iPad
  `min(88vw, …)` caps it to the screen. Verified in iPad Safari (simulator),
  portrait and landscape.

## 3. iPad / iPhone Safari: small text gets inflated

Readymag text widgets render `<p class="view-mode unstyled"
style="font-family:…;font-weight:…;font-size:8px">` (size inline, line height
from a class). `html` keeps `-webkit-text-size-adjust: auto`. On a zoomed page
(e.g. desktop layout at 0.8 on an 820-pt iPad) WebKit:

1. **text autosizing** inflates the computed line height (9px → 12.5px), and
2. a **minimum font size** bumps the computed font size (8px → 10px), because
   8 × 0.8 = 6.4 rendered is too small.

So `getComputedStyle()` on card text returns inflated values in Safari while
Chrome returns the specified ones. Copying those values into a pop-up gives
small-looking text with huge line gaps. Scaling with CSS `zoom` on the copy
makes it worse (Safari inflates again).

What works (verified on a local test page in iPad Safari: rendered 9.6/10.8,
same height as the reference; the old way was ~2× too tall):

- Read sizes with autosizing off for one synchronous moment:
  `R=document.documentElement; R.style.webkitTextSizeAdjust='none'; void R.offsetHeight;`
  … read … then restore the old value.
- Prefer the element's **inline** `style.fontSize` / `style.letterSpacing`
  (the specified values); take line height from the computed style read with
  autosizing off.
- Set explicit px (`specified × zoom × factor`) on the copy. Do not use CSS
  `zoom` on copied text.
- Put `-webkit-text-size-adjust:none; text-size-adjust:none` on the overlay.
- A wrapper `<p>` without its own font size makes every line at least its
  inherited 16px strut. Copy sizes onto the paragraph, not only its spans.

Slideshow images: the `.images-wrapper .image` slides may carry the picture as
inline `background-image`, as computed background only, or as an `<img>`
(progressive/LQIP loading on iOS). Read all three, or a cloned slideshow is
blank.

## 4. Testing on the iPad / iPhone Simulator

- `xcrun simctl boot <udid>`; `open -a Simulator` (needed for Device → Rotate
  via the menu). The simulator reaches the Mac's `localhost`.
- `simctl openurl` times out until Safari is running; launch Safari first
  (`simctl launch <udid> com.apple.mobilesafari`), terminate it if it hangs.
- The editor preview needs a Readymag login inside the simulator's Safari; the
  session does not survive a simulator reboot. Only the user can sign in.
- Typing a URL with the control tool's `text` breaks at `:`; use `openurl`.
- In the simulator the preview renders the **desktop** layout, and taps on the
  preview's eye button fall through to the page, so the Tablet/Mobile switch
  can't be used there. Real tablet-layout checks need a published (password-
  protected) project on a real device.
- When the real page is out of reach, reproduce the DOM in a local test page
  (zoomed container + Readymag-style `<p>`), serve it with
  `python3 -m http.server`, and open `http://localhost:<port>/` in the
  simulator. Measure with `getBoundingClientRect().height` and compare against
  a reference box with the expected px values.

## 5. Duplicating a page

- Pages panel → hover the page card → square icon = Duplicate (the ✕ next to
  it deletes; zoom in before clicking). The copy is inserted right after the
  source with a blank uri and is **visible**: hide it at once with
  `page.save({hidden: true, title: '…'}, {patch: true})`.
- Widgets keep the source order with fresh sequential ids, so map source →
  copy by index (check type and x per pair). Widgets with `is_global: true`
  (pinned on every page) are shared and not copied; never patch them for one
  page.
- Per-breakpoint overrides (`viewport_phone_portrait.{x,y,w,h,z,hidden,angle,
  animation,styles,blocksMeta}`) persist via widget patch save, and the phone
  `z` override is honoured by the preview.
- Hover-animation `trigger` arrays hold widget ids: re-point them to the copy's
  own ids, or the animation listens to nothing.

## 6. The live site's real widgets (no editor, no login)

- Public endpoint the published viewer itself calls:
  `https://<custom-domain>/api/viewer/project/<num_id>/widgets?pageId=<page _id>`.
  Page `_id`s come from `__RM_PROPS__` (`project.pages[]._id`). Found by
  watching the live page's network requests.
- Text widgets keep their words in `blocks[].text` (not `text`); animation
  objects come back with default fields stripped (same motion, fewer keys).
- A duplicate/spare project can differ from the live site (unpublished edits,
  older layouts). For "make it like the original", use the live endpoint.
  Map live → working copy by content (text blocks, image file name, video),
  shapes and slideshows by nearest desktop position.
- If shared text styles changed since, hidden text that sat under a photo can
  now poke out below it: check every card and hide what the original covered.

## 7. Making hover effects work on phones (tap toggle)

- A `hover` animation (photo `from_opacity 100 → 0` revealing text under it) never plays on touch. Fix that works:
  phone override `animation: []` on the media, then a tap toggle in `ci_body_bottom` that writes opacity rules into a
  generated `<style>` scoped by a class on `<html>` (inline styles on `.rmwidget` get reset by re-renders).
- Detect the phone layout from the page, not the window: `.page-content-container` `offsetWidth` is the canvas width
  (320 / 768 / 1024) in the preview and the viewer.
- Delegated click listeners on `window` only fire for taps in iOS Safari when the element looks clickable: give the
  tappable widgets `cursor:pointer`.
- Text that used to hide under a photo may be longer now (style changes): fit it on open — scale the text's inner
  `div` with `transform` and widen it by 1/scale so it re-wraps; `offsetTop`/`scrollHeight` are canvas px.
  Verified 12 Oct 2026 in the iPhone SE preview, 10 cards.

## 8. Writing project code safely

- **Another session (or person) may be editing `ci_body_bottom` at the same time.** Re-read it right before saving;
  apply your edits as string replacements on the current value and guard the write with a checksum of the value you
  merged onto. Never save a copy you read earlier.
- `fetch('http://127.0.0.1:…')` from the editor tab hangs (Chrome local-network-access), even with CORS headers.
  Send the code through the JS tool as string pieces instead.

## 9. Text links (nav)

- Links live in the text widget's `entityMap` (`data.type` Page/Anchor, `pageId`, `pageNum`, `pageUri`,
  `anchorPosition`, `linkStyle`), with separate copies in `viewport_tablet_portrait` / `viewport_phone_portrait`.
  Editing a link on desktop also rewrote the tablet copy (verified 12 Oct 2026).
- UI: double-click the box (edit mode) → double-click a word, shift-click to extend → link icon in the right toolbar →
  type a page number + Enter (or dropdown: Website URL / Type page number / Email / Phone / Anchor / Scroll to top /
  Go back) → pick the link style → click outside. Check the model after each change: re-linking part of an existing
  link dropped the other link in the same box once.
- An Anchor link is saved with the page you are editing from. For a pinned (every-page) nav, set it while on the
  target page. In Anchor mode the anchor = the spot you click on the page.
- Audit: walk `mag.getAllWidgets()` entityMaps and flag `pageId`s not in `mag.pages` (deleted pages leave dead links).
- Better than runtime fitting (12 Oct 2026, superseded the scale/translate version): lay the hidden-under-photo text
  out properly in the phone layout (measure content heights in the preview, grow the photo area where the text needs
  it, shift everything below), keep the text BELOW the photos in z, and let the code only fade all of a card's media
  together. Fallback without code = the original look. Widgets hidden on a breakpoint are not in the DOM at all on
  that breakpoint, so code can't clone them — they must stay visible (under something) if code needs them.
