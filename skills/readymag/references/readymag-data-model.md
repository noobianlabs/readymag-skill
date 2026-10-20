# Readymag Project Data Model — Annotated Reference

Two ways in: the **published HTML** (no auth) and the **editor Backbone model**
(auth). Same underlying data; the editor adds live widgets and write access.

## A. Published site — `__RM_PROPS__`

Every published Readymag page embeds the entire project model as
HTML-entity-encoded JSON:

```html
<script type="application/json" id="__RM_PROPS__" data-content="...">
```

```python
import json, re, html

with open('/tmp/rm_page.html') as f:
    raw = f.read()

m = re.search(r'id="__RM_PROPS__"[^>]*data-content="([^"]*)"', raw)
data = json.loads(html.unescape(m.group(1)))
project = data.get('project', data.get('mags', {}).get('mag', {}))
```

Fetch with `curl -sL <url> -o /tmp/rm_page.html`. The blob is 50–90KB — save
to a file and parse; don't read it inline (terminal output truncates).
Any page URL returns the complete project model.

### Project-level fields

| Field | Example | Notes |
|-------|---------|-------|
| `num_id` | `1234567` | Published project ID |
| `uri` | `"my-site"` | Project slug |
| `desktopWidth` / `phoneWidth` | `1024` / `320` | Canvas widths |
| `pages_count` | `6` | Page count |
| `is_published` | `true` | |
| `last_published` vs `last_changed` | timestamps | `last_changed` later = unpublished edits exist |
| `usedWidgetTypes` | `["background","text","picture","shape",...]` | Widget inventory |
| `hasAnimations` | `true` | Hover and/or scroll animations present |

### Pages array — viewport detection

```json
{
  "num": 1, "uri": "home", "type": "scroll", "height": 1287,
  "viewport_phone_portrait":  { "enabled": true,  "height": 1468, "type": "scroll" },
  "viewport_tablet_portrait": { "enabled": false }
}
```

Three states per breakpoint: `enabled: true` (active layout), `enabled: false`
(created then disabled), **key absent** (never created — clean slate).

## B. Editor model — `window.RM`

On `https://my.readymag.com/edit/<id>/<pageNum>/`:

```js
const m = window.RM.constructorRouter.mag;
m.pages.models                        // page models
m.pages.models[0].widgets.models      // ALL widget models, loaded
```

- `page.attributes` → `num, uri, height, viewport_default,
  viewport_tablet_portrait, viewport_phone_portrait`
- `widget.attributes` → `x, y, w, h, z, type, hidden, fixed_position,
  pack_id, animation, triggers, is_full_width, is_locked` plus per-breakpoint
  overrides in `viewport_tablet_portrait` / `viewport_phone_portrait`
- Effective geometry = viewport override merged over base attributes.
- Per-breakpoint override objects can carry `x, y, w, h, z, pack_id, angle,
  animation, blocksMeta, styles, entityMap` — `animation: []` in an override
  disables the desktop animation on that breakpoint only.
- Crop attributes on pictures are in source-image pixels and
  breakpoint-independent — resizing needs no crop changes.
- On `/edit/<id>/preview/` the model is `constructorRouter.previewMag`;
  **widgets are NOT loaded** (`page.widgets.models.length === 0`).

### `fixed_position` anchor codes

`n` top-center · `nw` top-left · `ne` top-right · `c` center (NOT
canvas-relative — overflow math against canvas width gives false positives).
Screen x of a fixed widget = `x / 1024` × window width, at every width.

### Write patterns

```js
// Widget: geometry, animation, overrides — persists
w.set({...}); await w.save({...}, {patch: true});   // PATCH /api/widget/<id>

// Page attributes — persists ONLY with patch:true and specific attrs
await page.save({viewport_tablet_portrait: v}, {patch: true});

// z — returns success but does NOT persist. Layer order for
// "Fixed on all pages" lives in the Layers panel group order (drag to fix).
```

Getting data out: printing large JSON truncates, and the JS-tool content
guard blocks results containing cookie/query-string data (`viewport_*`
objects embed screenshot URLs). Print scalar fields only, or download:

```js
const a = document.createElement('a');
a.href = URL.createObjectURL(new Blob([JSON.stringify(backup)]));
a.download = 'backup.json'; a.click();   // lands in ~/Downloads
```

## C. Animation objects

- Types seen: `scroll` (displacement driven by real scroll; `dx` at speed 1 ≈
  1px per scrolled px) and `hover` (e.g. `from_opacity: 0` fade-ins — dead on
  touch devices).
- A widget's resting `x/y` is the animation's **start** position; rendered
  position = resting + displacement. Collage pieces routinely rest hundreds
  of px off-canvas by design.
- Scroll triggers are computed at page load and never recomputed on resize.

## D. URLs

| What | Where | Auth |
|------|-------|------|
| Published site | custom domain or `user.readymag.com/<slug>` | No |
| Editor | `my.readymag.com/edit/<id>/<pageNum>/` | Yes |
| Editor preview | `my.readymag.com/edit/<id>/preview/` | Yes |

Editor project ID ≠ published `num_id` when working on a duplicate. Confirm
which project is open before editing.
