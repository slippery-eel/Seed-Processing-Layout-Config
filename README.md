# Seed Processing Facility — Layout Study

An interactive, to-scale layout tool for a seed-processing facility, in a single self-contained
HTML file (`floorplan.html`). The site is a fixed right-angle trapezoid shell; everything inside
it is a movable object:

- **Zones** and **boundary areas** (anterooms, pass box, WIP/buffer racking) — rectangles you can
  drag, corner-resize, or size by typing width/height (feet) in the **Zones & Areas** table.
- **Fixtures** — point features (eyewash, sinks, dust drops, fume exhaust) and line features
  (trench drains, roll-up / tote / cart doors) — drag to reposition (no resize).
- **Process equipment** — draggable / rotatable / resizable machines, coloured by their zone.

You can zoom, toggle a machine-clearance overlay, switch to a decluttered **Presentation** view,
save multiple named arrangements, or export the current view as a PNG (with a legend).

Repurposed from an apartment interior-design tool. The reference scheme is the **"All at Peckham"
(Config 1)** option from `docs/facility-layouts.html` in the companion `flow-sim` repo — the whole
process line in one owned shell, wet front end in a prefab pole barn, priming and pelleting
in-house.

No build step, no dependencies, no server. Open `floorplan.html` directly in a browser.

---

## Quick start

Double-click `floorplan.html`, or open it via `File → Open` in any modern browser.

| Action | How |
|---|---|
| Move anything | Click and drag it (equipment, a zone, an area, a fixture) |
| Select | Click it, or click its row in the Zones & Areas / Equipment tables |
| Resize a zone or area | Drag a corner handle, or type W/H (feet) in the Zones & Areas table |
| Resize equipment | Type W/H (inches) in the Equipment Schedule |
| Rotate equipment | Drag the ring (`●`) above a selected piece; `R` / `Shift+R` for ±15° (equipment only) |
| Nudge position | Arrow keys (0.25 ft; hold `Shift` for 1 ft) |
| Recolour | Click the swatch next to an item's name (zone colours + neutrals) |
| Zoom | The **Zoom** slider (0.5×–3×) — the site is 180 × 140 ft, machines are 2–8 ft |
| Show required clearances | **Clearance Overlay** button, then select a machine |
| Declutter for a screenshot | **Presentation** button (see below) — remembered per browser |
| Export the current view | **Export PNG** — 3×, with a legend band; honours Presentation mode |
| Save / switch arrangements | The **Layouts** row (saves zone/area geometry too) |

---

## Preset layouts

Three hand-built arrangements ship in the file and appear in the **Layouts** dropdown on every
open (including via `file://`); they're seeded into local storage on load, only if their stable
id is missing, so deleting one just brings it back on the next load. **Reset Layout** is
unaffected — it still restores the code defaults, not a preset. All three respect the
`D1 → A → B1 → D2 → C → B2 → E` process order; they differ in how the zones are packed into the
trapezoid:

| Layout | Idea |
|---|---|
| **Flow A — Process Horseshoe** | Clockwise wrap: intake + wet across the narrow top, dusty B1 down the full-height left edge, cold room + clean lab through the middle, pelleting on the right, finishing/dispatch as a full-width band along the wide bottom feeding the shared roll-up door, office in the bottom-right wedge. |
| **Flow B — Diagonal Throughput Line** | One material path stepping from the narrow top-left corner down toward the wide bottom-right, B1/B2 as bays on the same wall for shared dust extraction, a single main aisle, office off the goods line in the bottom-left corner. |
| **Flow C — Clean Core (hygiene-zoned)** | D2 → C → E form a protected central spine ringed by the anterooms and the two-door pass box; the wet/dusty zones A, B1, B2 sit on the top and side perimeter with their dust/drain fixtures on the outer walls; office/gowning at the core's left entrance. |

Edit them like any layout and **Save**, or **Save As New** to branch one.

---

## Presentation mode

The **Presentation** toolbar button toggles a decluttered view for screenshots and slides
(the choice is remembered per browser). It changes only labels, never geometry:

| | Detailed (default, for editing) | Presentation |
|---|---|---|
| Zone / area label | `id · full name` + sqft, top-left of the rect | `id` + short tag (e.g. `A  WET`), top-left; areas show the tag only |
| Equipment | Full name wrapped on the box | A **number** on the box + a short name **below** it (staggered so packed rows don't collide) |
| Fixture | Full label, centred above the marker | Short label (`CART DOOR`, `DUST DROP`, …), centred above the marker |

**Export PNG** renders at 3× and appends a **legend band**: the zone colour key with live sqft,
and a numbered `1 … 27` equipment list (grouped by zone) whose numbers match the boxes — so a
Presentation-mode export is self-describing. The file is named `seed-facility-layout-presentation.png`
or `-detailed.png` after the active mode.

Short names live in `short:` fields on each `AREAS`, `FIXTURES` and `EQUIPMENT` entry; equipment
numbers are assigned in the Equipment Schedule's display order (A 1–5, D1 6, B1 7–10, D2 11,
C 12–17, B2 18–21, E 22–25, OFF 26–27). One rotated machine (`pelletDryer`, 90°) shows its
name beside the box rather than below.

---

## The site

A **right-angle trapezoid**, right angle on the left (left edge vertical):

- Left edge (vertical): **140 ft** · top parallel edge: **65 ft** · bottom parallel edge: **180 ft**
- Vertices in feet: `(0,0) → (65,0) → (180,140) → (0,140)`
- Area ≈ **17,150 sqft**. `SCALE = 5` px/ft, `PAD = 40`; a 10 ft grid, heavier every 50 ft,
  clipped to the trapezoid.

## Zones & areas

`AREAS[]` in `floorplan.html` — one array of editable rectangles: `{id, kind, name, note, color,
x, y, w, h}` in **feet** (top-left origin; the engine works in centre `cx`/`cy` like equipment).
`kind:"zone"` also carries a `target` sqft (the doc's Config-1 program). `kind:"area"` covers the
boundary rooms — the A→B1 and D2→C anterooms, the C→B2 pass box, and the intake / dried-lot / WIP
racking blocks. Every rectangle is draggable, corner-resizable, and editable via the **Zones &
Areas** side table.

Zones sit in process order with no backtrack — `D1 → A → B1 → D2 → C → B2 → E`, with B1 and B2
flanking C. Equipment keeps an explicit `zone` string that drives its default colour and schedule
grouping — moving a zone rectangle does **not** reassign equipment.

| Zone | Name | Target sqft | Colour |
|---|---|---|---|
| A | Wet / Chemical — prefab pole barn (trench drains, fume extraction, uninsulated) | 3,000 | `#3b7bbf` |
| D1 | Berry intake / holding — climate-controlled 55–85 °F, **not refrigerated** | ~400 | `#d8c27a` |
| B1 | Dry & Dusty, pre-priming — NFPA 61/660 dust collection, negative pressure | 1,400 | `#c79a4b` |
| D2 | Seed-holding **cold room** — real insulated cold room, strip curtains | 330 | `#6bb8c7` |
| C | Clean Lab & Priming — controlled temp/RH, gowning anteroom, no pollen-lab share | 1,500 | `#5c8a5c` |
| B2 | Dry & Dusty, pelleting suite — 2nd NFPA dust drop | 1,050 | `#c1552c` |
| E | Finishing / QA / Dispatch — GMP dry-warehouse hygiene | 1,330 | `#8a5c8a` |
| OFF | Office / Gowning | 390 | `#8a8f98` |

The Zones panel (below the schedule) shows the legend plus a live readout of drawn zone area vs
the 17,150 sqft site.

## Equipment

`EQUIPMENT[]` in `floorplan.html`. Same object schema as the original furniture — `{id, label, w,
h, cx, cy, rot, color}` with **`w`/`h` in inches**, `cx`/`cy` the centre in feet — plus a `zone`
field that drives the default colour, and an `est` flag for items with no vendor dimension in the
source doc (shown as an `est` badge in the schedule). One unit of each machine (Config 1, Phase 1).
Footprints are the doc's §1.4 values (L×W ft × 12).

| Zone | Equipment |
|---|---|
| A | Berry Colour Sorter *(est)* · Seed Extraction · DRL Drum Disinfect · DRP-4000 Centrifuge · DRM-4100 Dryer |
| D1 | Berry bin / tote racking |
| B1 | Air Screen · Density Grading Table · Seed Color Sorter · Air Column |
| D2 | Cold-room pallet racking |
| C | Osmotic Priming Unit *(est)* · Post-priming Re-dryer *(est)* · Water-activity & Priming QA bench *(est)* · Germ Chamber · Growth Chamber · Thermogradient Table *(est)* |
| B2 | Powder Blender · Pelleting Pan · Pellet Dryer · Dry Sizer |
| E | Count / Weigh / Package *(est)* · Final QA Bench *(est)* · Retain-sample Storage *(est)* · Finished-goods racking |
| OFF | Gowning benches · Site office desk |

## Fixtures

`FIXTURES[]` — point and line features, **draggable to reposition** (no resize/rotate).
`{id, kind:"fixture", fx:"point"|"hline"|"vline", cx, cy, len?, label, color}`:

- One grade-level **roll-up door** (shared inbound berries + outbound finished goods)
- **Trench drains**, **fume exhaust**, **eyewash / safety shower**
- Two **NFPA 61/660 dust-collection drops**, two **handwash sinks**
- **Tote door** (berries in), **B1→D2 strip-curtain cart door**, **B2→E cart / pallet door**

(The A→B1 and D2→C anterooms and the C→B2 pass box are `AREAS`, above — they resize.)

## Clearance overlay

**Clearance Overlay** + a selected machine draws the doc's §1.4 clearances: 36 in of clear floor
on the access sides, plus a full machine-depth **pull-out** zone for the DRM-4100, DRP-4000,
priming unit and re-dryer. Shown unrotated for machines at a non-zero angle.

---

## Architecture

Single file: `<style>` → HTML body (toolbar rows + the `#stage` / schedule two-column grid) → one
`<script>` IIFE. `#stage` stacks three absolute layers: `#floorplanSvg` (static grid + perimeter),
`#areaLayer` (zone / area / fixture DIVs), `#furnitureLayer` (equipment DIVs), `#labelsSvg`
(clearance overlay). Layers are `pointer-events:none` with `:auto` items, so a click falls through
empty space to whatever is beneath, and equipment (later in the DOM) always wins over a zone.

**Unified item model.** Everything movable lives in `state` / `order` and shares
`startDrag`/`select`/`handleKey`/snapshot. A `kind` field branches `buildItemEl` and `render`:
no kind = equipment (inches, rotate handle); `"zone"`/`"area"` = feet rectangle with four
`.resize-handle` corners (`startResize`); `"fixture"` = point/line, move-only.

Removed from the apartment original: the Sun View daylight simulation and the wall-outlet editor.

## Persistence (localStorage)

| Key | Holds |
|---|---|
| `seed-facility-dims-v1` | `{itemId: {w?, h?}}` global **equipment** size overrides |
| `seed-facility-slots-v2` | `{slots: [...], activeSlotId}` — saved arrangements; each item stores `cx/cy/rot/color`, plus `w/h` for zones & areas (their size is per-layout, not global). The three `preset-*` layouts (see **Preset layouts**) are seeded here on load if absent. |

Per-browser, local-only. Zoom and clearance-overlay state are session-only.

## File map

- **`floorplan.html`** — the entire application.
- **`plan.md`** — short note on the repurpose from the apartment tool.
- **`README.md`** — this file.
