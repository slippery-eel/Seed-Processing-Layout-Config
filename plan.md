# Plan / changelog

## Repurpose: apartment interior-design tool → seed-processing facility layout tool

`floorplan.html` began as a to-scale apartment floorplan with draggable furniture. It was
retargeted at a **seed-processing facility layout** based on the "All at Peckham" (Config 1)
scheme in the companion `flow-sim` repo's `docs/facility-layouts.html`.

Done in four iterations, each left the file browser-openable and was checked before the next:

1. **Site shell + zones.** Removed the apartment geometry, the Sun View daylight simulation, and
   the wall-outlet editor. New `SCALE`/`PAD` for a 180 × 140 ft site. Drew the right-angle
   trapezoid (`SITE_POLY`), a 10/50 ft grid clipped to it, the 8 colour-coded `ZONES` in process
   order (`D1→A→B1→D2→C→B2→E`, B1/B2 flanking C), a zone legend and a drawn-vs-site sqft readout.

2. **Equipment layer.** `DEFAULTS` → `EQUIPMENT[]` from the doc's §1.4 footprint schedule, one
   unit of each machine, each with a `zone` (drives its default colour) and an `est` flag for
   items with no vendor dimension. "Furniture Index" → "Equipment Schedule", rows grouped by zone
   with coloured headers and `est` badges. Recolour palette → the 8 zone colours + neutrals.
   `localStorage` keys → `seed-facility-*`.

3. **Fixtures & boundary features.** `FIXTURES[]` + a renderer: roll-up door, trench drains, fume
   exhaust, eyewash/safety shower, two NFPA dust drops, handwash sinks, the six zone-boundary
   transfer features (anterooms, strip-curtain door, pass box, cart doors), and WIP/buffer
   racking.

4. **Zoom, clearance overlay, export, rename.** A 0.5×–3× zoom control (CSS transform on `#stage`;
   drag math divided by the zoom). A "Clearance Overlay" toggle drawing the §1.4 clearances
   (36 in access + a machine-depth pull-out for the DRM-4100 / DRP-4000 / priming unit /
   re-dryer). PNG export dropped the removed sun/outlet layers and renames the download.
   Page, `README.md` and this file retitled.

The reusable engine — drag / rotate / resize, the schedule table with live number fields, the
colour popover, the save-slot system, the foreignObject-free PNG export — was kept intact; only
its data changed from furniture to process equipment.

## 5. Editable zones & areas

Zones and the rectangular fixtures were static SVG; now they're first-class movable objects.

- `ZONES` polygons → `AREAS` **rectangles** (`kind:"zone"|"area"`, `x/y/w/h` feet). All zones are
  plain rectangles; the site stays a trapezoid.
- New `#areaLayer` DIV layer (below `#furnitureLayer`); `buildItemEl` dispatches to
  `buildAreaEl` / `buildFixtureEl` / `buildEquipEl`; `render` branches on `kind`.
- Zones & areas: drag to move, four corner `.resize-handle`s (`startResize` — opposite corner
  anchored, zoom-corrected, min 2 ft, snap 0.5 ft), and a new **Zones & Areas** side table with
  W/H (ft) inputs + swatch recolour. Live sqft label + legend + readout (`renderZoneStats`).
- Point/line fixtures (`kind:"fixture"`, `fx` + `len`) become draggable DIVs, move-only, with a
  compact side list.
- Snapshots carry `w/h` for kind items; `SLOTS_STORAGE_KEY` bumped to `-v2`; PNG export gained
  `drawAreaLayer`. Equipment behaviour and its `zone` assignment are unchanged (no auto-reassign
  on drop).

## 6. Shipped preset layouts

Three cohesive, process-flow-respecting arrangements baked into `floorplan.html` as
`PRESET_SLOTS` and seeded into the save-slot list on load (`seedPresetSlots()`, added only if the
stable `preset-*` id is absent, so a deleted preset re-seeds next load). Each `items` map is a
full snapshot (`cx/cy/rot` per equipment, `+w/h` for zones/areas, `cx/cy` for fixtures) — no
engine change; `doLoadSlot` → `applySnapshotToState` already rebuilds everything.

- **Flow A — Process Horseshoe**: clockwise wrap, wet across the top, dusty B1 down the left,
  priming centre, pelleting right, finishing/dispatch full-width along the bottom.
- **Flow B — Diagonal Throughput Line**: one aisle stepping from the narrow top-left to the wide
  bottom-right; B1/B2 as same-wall bays; office off the goods line.
- **Flow C — Clean Core**: D2→C→E as a protected central spine ringed by anterooms + the pass
  box; A/B1/B2 on the perimeter with dust/drain fixtures on the outer walls.

`trenchDrain` default `len` cut 82 → 50 ft (snapshots can't override a fixture's line length, and
82 ft overran every zone). **Reset Layout** still restores the code `AREAS`/`FIXTURES`/`EQUIPMENT`
defaults, not a preset.

## 7. Label readability + Presentation mode

On-canvas text was unusable for screenshots (equipment names `nowrap` at 10.5px streaked across
the plan; fixture labels shot sideways; zone names sat centred under the machines). Added:

- A `labelMode` (`"detailed"` / `"present"`) with a **Presentation** toolbar button, persisted to
  `localStorage["seed-facility-labelmode"]`; `applyLabelMode()` re-renders on toggle and on load.
- `short:` fields on every `AREAS`, `FIXTURES` and `EQUIPMENT` entry; an `EQUIPMENT` post-pass
  assigns `tag` numbers in the schedule's zone-group order (A 1–5 … OFF 26–27).
- Labels reworked in `buildAreaEl` / `buildFixtureEl` / `buildEquipEl` + `render`:
  zone/area labels moved to the rect's **top-left corner** (both modes); fixture labels **centred
  above** the marker; equipment gets a `.lbl` (wrapped full name, detailed), a `.lbl.tagnum`
  (number, present) and a `.lbl-below` (short name below the box, present, fixed 66 px width,
  staggered into 3 tiers by left-to-right order within the zone so packed rows don't collide).
- PNG export mirrors all of it: `drawAreaLayer` / `drawFurnitureLayer` branch on `labelMode` and
  use a `wrapText` helper; `scale` 2 → 3; a `drawLegend()` band is appended below the plan
  (`LEGEND_H` px taller canvas) — zone colour key with live sqft + a 3-column numbered equipment
  list; download name suffixed `-presentation` / `-detailed`.

Interactions (drag / rotate / resize / schedule) untouched. `pelletDryer` (rot 90°) shows its
name beside the box rather than below — acceptable, noted in the README.
