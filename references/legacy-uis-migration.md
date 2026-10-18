# Legacy UIS to MalodyV Migration

For a complete port, inventory the complete selected skin graph; for a local fix, follow only affected includes, branches, assets and owners. Start with the three-phase procedure; use Function mapping and Units and animation conversion as lookups. Validation/deployment requirements apply to the changed scope and requested actions.

This reference converts the historical UIS format documented for Malody versions
before 4.3.7 into a current MalodyV skin graph. It is intentionally scoped to
Key and other straight-line note modes documented by the legacy manuals,
including IIDX/pop'n/DDR-like layouts.

The historical source basis was the supplied official `UIS 手册.pdf` and `UIS动画手册.pdf` (not bundled here; original revision/URL is unrecorded).
They document the legacy syntax and applicability, not the current host
runtime. Current Lua/Composer names below are therefore routed through the
existing official and target-observed references in this skill.

The default goal is **visual equivalence**: preserve the old skin's visible
layout, assets, layering, note construction, state feedback, and animation
timing as closely as the target host permits. Do not silently claim gameplay,
judgment, input, score, or replay equivalence when the target contract has not
been proved.

Use this evidence label on every mapping that affects runtime behavior:

```text
official | target | probe | inferred | unknown
```

`official` is a documented API or legacy syntax fact. `target` is observed in
the supplied/new skin or host artifact. `probe` is a controlled exact-host
observation. `inferred` is a reversible visual implementation choice.
`unknown` blocks a strong compatibility claim and names the next probe.

## Three-phase migration procedure

Use this procedure for standard 437 straight-line/key skins such as 4K–8K.
The workflow is intentionally visual-first. Judgment, score, timing, input,
Full Combo, and replay equivalence require their own target contract and probe.

### Phase 1 — Inventory and confirm source facts

Create a source-to-target inventory before editing. Record:

- legacy version, selected mode, key count, lane order, platform, resolution,
  aspect ratio, and reference screenshots;
- source skin root, target MalodyV skin root, active deployed copy, and cache path;
- every `.mui`/UIS entry, `@include` edge, conditional branch, variable, and
  duplicate property after expansion;
- each node's owner: static UI, lane/key, note, hit/judge feedback, animation,
  input/touch, data display, or unsupported/unknown;
- units, anchors, layers, base sizes, textures, frames, Scale3/Scale9 data,
  and the target `info.asm`, Composer IDs, Lua callbacks, and settings fields.

Pause only conversions that depend on an unknown active entry, mode, key count,
target host/version, or visual owner. Continue independent inventory and source
measurements; do not claim compatibility for the unresolved branch.

### Phase 2 — Calculate target parameters

Build a conversion ledger instead of copying legacy numbers directly. For each
visual element, record the source expression, source unit, target owner, target
unit, normalized screenshot ratio, target scene-scale input, asset canvas and
alpha bounding box, judgment-position coordinate if relevant, and evidence class.

Use these rules:

1. Convert legacy relative units with the applicable legacy manual. A legacy
   bare number is not automatically a MalodyV Unit value.
2. Normalize measurements by screen height or playfield height before comparing
   devices. Compare lane spacing, playfield width, and visible asset size as
   ratios, not only absolute pixels.
3. Derive scene scale from lane/playfield ratios. Keep scene-scale correction
   separate from asset visible-size correction.
4. Use transparent padding only for visible artwork size or visual offset; it
   does not move the logical lane or judgment position.
5. Derive judgment-position mappings from at least two target measurements and
   label the result `probe` or `inferred` until a second target condition confirms
   it.

Required ledger shape:

```text
legacy node/property:
source expression/unit/anchor/layer:
target owner (ASM/Composer/Lua/asset/native):
target field or resource:
target unit/index/coordinate root:
normalized measurement:
scene-scale input:
asset canvas/alpha bounds:
judgment-position mapping:
evidence: official | target | probe | inferred | unknown
validation and next probe:
```

### Phase 3 — Migrate, deploy, and validate

Implement in this order: asset inventory, static layout, lane/key and note
templates, text/data nodes, animations, event-driven feedback, then exact-host
validation. Before overwriting any source or deployed file, create a timestamped
backup and record whether rollback covers source, deployment, or both.

Deploy only when explicitly requested. Copy only confirmed files, verify target
hashes, refresh only the confirmed resource cache, and preserve unrelated user
changes. For a complete port, the acceptance pass covers tap, hold, chord, key released/pressed,
hit/judge feedback, score/combo, pause/resume, retry/reload, a second
resolution/aspect ratio, cache refresh, rollback, and exact-host visible output.
For a local fix, select affected rows; unavailable host checks remain unverified,
not a reason to abandon independent local work.

## Standard-key mapping guide

Use the following as a category-level map. The exact current owner and API must
be confirmed for the target MalodyV version and mode.

| Legacy category | Typical legacy evidence | MalodyV target candidate | Migration boundary |
|---|---|---|---|
| Ordinary image | `type=0`, `tex`, `size` | Composer image/sprite module | Preserve asset, base size, anchor, layer, and visibility owner. |
| Text/score | `type=1`, `{title}`, `{artist}`, score fields | Composer text/number module plus authoritative Basic/Play values | Keep source placeholders separate from formatted runtime values. |
| Solid/shape | `type=2`, `color` | Composer solid/shape module or verified solid asset | Validate RGB, Alpha, unit, and coordinate root. |
| Frame animation | `type=3`, `frame`, `interval` | Composer frame animation or owned Lua animation state | Define first-frame, hidden-state, reset, and retry behavior. |
| Scale3/Scale9 | `type=4/5`, `part`, `size2`, `rect` | Composer stretch/slice/mask module or verified `SetSlice` path | Preserve fixed ends, stretch axis, source-pixel rect, and orientation. |
| Lane/key | `key-N`, `tex`, `tex2` | Lane-specific Composer/key modules plus input or Trigger state | Confirm lane index, released/pressed ownership, and reset. |
| Tap/hold/chord note | `note-N`, `tex`, hold `type`, `tex2/3/4/5`, `type2` | Native or Composer note templates, lane mapping, and target assets | Separate tap, hold head/body/tail, chord, and lane art. Native position fields require a probe. |
| Hit/press/judge feedback | `hit-N`, `press-N`, `judge-N` | Hit callbacks, Trigger state, frame animation, or judgment display | Do not infer judgment enum or duration from legacy syntax alone. |
| Score/combo/HP/progress | `score-*`, `progress`, `hp` | Composer text/number/progress nodes and authoritative runtime values | Test zero, first event, misses, large values, and retry. |
| Pause/touch overlay | pause/touch nodes and states | Overlay Composer nodes, input callbacks, or target UI settings | Validate pause ownership, touch state, and resume reset. |

### What must not be assumed

- MUI/UIS position and size expressions are not automatically valid ASM or
  Composer values.
- Generic `X/Y/DX/DY` fields do not prove that native note instances accept
  per-lane or per-instance displacement.
- Scene scale can affect lanes, playfield, and native artwork together; it may
  not independently resize note artwork.
- Transparent padding changes visible geometry, not logical judgment position,
  lane ownership, or input mapping.
- A formula measured from one skin, resolution, platform, or host version is
  scoped evidence, not a universal engine contract.

## Conversion worksheet

Copy this worksheet for each migrated skin and retain it with the source and
deployment evidence:

```text
Skin/source:
Legacy version and mode:
Target MalodyV version/platform/mode:
Key count and lane order:
Reference resolution/aspect ratio:
Source root / target root / active deployed copy:
Cache path and reload procedure:

Legacy entry/includes/conditions:
Variables and duplicate-property precedence:

Element mapping:
  legacy name | category | lane/index | source expression/unit/anchor/layer |
  target owner | target field/resource | target unit/index | normalized ratio |
  scene-scale input | asset canvas/alpha bounds | judgment mapping | evidence |

Animations:
  legacy group/inline motion | source timing/properties | target animate/API | evidence

Runtime ownership:
  Init:
  Update:
  OnHit:
  OnInput:
  retry/reload/reset:

Acceptance:
  tap:
  hold head/body/tail:
  chord:
  key released/pressed:
  hit/press/judge:
  score/combo/accuracy:
  HP/progress/musicbar:
  pause/resume/touch:
  second resolution/aspect:
  cache refresh:
  exact-host evidence:
  backup/hashes/rollback:
```

## BeatTube case study (scoped target/probe evidence)

The historical source/target paths, geometry, scene-scale observations, and judgment calibration are preserved in the optional [BeatTube case study](legacy-beattube-case-study.md). Do not load or copy its constants for an unrelated migration.

## Migration acceptance checklist

Use applicable rows after implementation; deployed-copy/cache/hash rows apply only when deployment occurred. A static parse or successful file copy
does not prove visual or gameplay equivalence.

1. Confirm the exact legacy entry, target skin root, MalodyV version, platform,
   mode, key count, and active deployed copy.
2. Parse all `.mui`/UIS includes, conditions, variables, collections, and
   duplicate properties; retain the source ledger.
3. Verify every target Composer ID, ASM field, Lua callback, asset path, frame,
   slice, canvas size, and alpha bounding box.
4. Compare static lanes, playfield, tap notes, hold head/body/tail, chords,
   key released/pressed states, and layer order.
5. Compare hit/press/judge feedback, score/combo/accuracy, HP/progress/musicbar,
   pause/resume, and touch overlays where present.
6. Check retry, reload, repeated hits, holds, reset, and animation drift.
7. Check a second resolution/aspect ratio and remeasure scene scale rather than
   reusing case-study constants.
8. Refresh only the confirmed cache, verify destination hashes, and test the
   rollback path.
9. Record exact-host version/platform/mode, visible output, relevant log stage,
   evidence class, remaining uncertainty, and acceptance result.

## Legacy inventory

Before editing, make a source inventory and preserve the original files outside
the deployment directory. Record the active skin root, target MalodyV version,
mode, key count, display resolution, and source/deployment paths.

1. Locate every `.mui`/UIS entry and follow `@include` edges.
2. Expand conditional includes and 4.0 conditional text sections for the
   selected mode/version; retain the original condition in the ledger.
3. Resolve variables, comments, repeated properties, and node collections such
   as `_sprite-[1,2,3]` or `_sprite-[1-3]`.
4. Classify each node as ordinary, lane-specific, note/gameplay, animation,
   touch input, or unsupported/unknown.
5. Match every referenced texture, frame, Scale3 part, Scale9 rect, font-like
   number sequence, and animation group to an asset and a visible owner.

The legacy parser is line-oriented: one property per line, tab-indented under
the node name, with later duplicate properties overriding earlier ones. Names
are case-sensitive, limited to 40 characters, and should remain ASCII in
generated files because the old manuals warn that unexpected full-width
characters can crash the old loader.

## Function mapping

| Legacy UIS | MalodyV target path | Migration rule |
|---|---|---|
| Ordinary `type=0` image node, `tex`, `size` | Composer image/sprite module; `Module:Find` in Lua when dynamic access is needed | Preserve asset, base size, anchor, layer, and Alpha. Use Lua only for per-frame or event-driven changes. |
| `type=1` text and `{title}`, `{artist}`, `{creator}`, `{version}` | Composer text module plus `Text`, or a Lua display adapter using `Game:ChartInfo` | Keep source text placeholders distinct from formatted runtime values; verify empty/long metadata. |
| `type=2` solid rectangle and `color` | Composer solid/shape module, or a solid asset when no target shape contract is available | Treat color conversion as visual; verify alpha and coordinate units on host. |
| `type=3` frame animation, `frame`, `interval` | Composer frame-animation module with `Play`/`Stop`, or cloned/owned Lua animation state | Define first-frame and hidden-state behavior explicitly; never assume `Stop()` hides the node. |
| `type=4` Scale3, `part`, `size2` | Composer stretch/slice component or `SetSlice` where target version supports it | Preserve fixed ends and stretch axis. If no target slice contract exists, use segmented assets and mark approximate. |
| `type=5` Scale9, `rect` | Composer 9-slice/mask component or `SetSlice(left, top, right, bottom)` on supported hosts | Convert source-pixel rects to target slice edges; verify orientation and texture coordinate order. |
| `note-N`, `tex`, hold `type`, `tex2/3/4/5`, `type2` | Lane-specific Composer Note template/modules, with Lua/Trigger only for dynamic feedback | Separate tap, hold head/body/tail, chord, and lane instance art. Do not reuse ordinary UI duplicate-ID rules for Note templates. |
| `key-N`, `tex`, `tex2` | Lane-specific key modules plus input/Trigger state | Map released/pressed art first; validate input state and retry reset on the target host. |
| `hit-N`, `press-N`, `judge-N` | Frame/animation modules, hit callbacks, and judgment display modules | Preserve event-to-visual ownership, but do not infer judgment enums or duration from legacy syntax alone. |
| `score-combo`, `score-score`, `score-acc` | Composer number/text modules and authoritative Basic/Play values | Keep source values separate from formatted text; test zero, first event, misses, large values, and retry. |
| `score-hp`, `progress` | Composer bar/mask/point modules or a Lua-driven bar | Preserve direction, fill/clipping behavior, count, and point spacing only after target verification. |
| `musicbar` | Composer music/progress bar or a target-observed Lua display | Record whether `count` means segment count, statistic divisions, or a visual-only subdivision. |
| `pause` | Composer pause-area node or target input/Trigger boundary | A visible pause button and its touch region are separate acceptance items. |
| `touch` | Composer input regions/Triggers or target-mode input callback | Preserve rectangle, circle, triangle, and lane mapping geometry only when the target mode exposes the same path. |
| `@include`, variables, collections | File/module decomposition, Composer variables, `Clone`, or generated per-lane entries | Expand deterministically before mapping; retain the source expression in the migration ledger. |

## Units and animation conversion

### Coordinates and sizes

Legacy UIS accepts relative units, percentages, and `px`, with simple addition
and subtraction. Normalize every value into a named target coordinate space:

```text
legacy expression -> evaluated value at reference resolution -> target X/Y/Width/Height -> anchor/layer
```

Record the reference resolution, scene scale, origin, anchor, and whether the
legacy value describes a top-left, center, bottom-left, or lane-relative point.
Do not copy raw numbers without this record. Re-check at a second aspect ratio
and at the target's actual scene scale.

MalodyV Composer units (ModuleParamUnit enum, editor-confirmed 2026-08-07) differ
from UIS units:

- `Percent`: fraction of the current screen dimension (width for Width/X, height for Height/Y).
- `Unit`: `1/1080` of screen height on the 1920×1080 reference canvas (1 logical pixel at 1080p); 100% of screen height = 1080 units. Evidence: Mutsumi `mutsumi.lua` KPS `logW = Game:Width()` = 1920 at 1080p with `kpsbarbg.Width = 300` Unit and `stepX = logW / physW` (300 Unit = 300 logical px); the Free Mode official example spans 1080 units vertically.
- `Px`: physical pixels.

Legacy UIS relative units are `1 = window height / 720`. Conversion to MalodyV:

- UIS percentage → `Percent`: 1:1 (width and height).
- UIS relative `N` → `Unit`: `N × 1080/720 = N × 1.5` (height values and y offsets).
- UIS `px` → `Px`: 1:1.

Worked example (1173_BeatTube_Skin): `_buttom` `size=100%,60` → Width 100 Percent, Height 90 Unit; `score-hp` `size=30.234%,15` → Width 30.234 Percent, Height 22.5 Unit; `score-hp` `pos=97.95%,99%-6.5` → dy = -9.75 Unit. Verified in the beattube port (2026-08-07): a naive 1:1 Unit copy still under-scales the elements; the ×1.5 conversion restores the original 8.33% and 2.08% of screen height. Do not treat legacy bare numbers as MalodyV Unit values.

### Properties

Use the closest current property owner:

| Legacy property/operation | Current API candidate | Important constraint |
|---|---|---|
| `opacity`, `Alpha`-like visibility | `Alpha`, `DoAlpha` | Legacy opacity is `0–250`; current `Alpha` is `0–100`. Convert with `current = legacy / 2.5`, then verify rounding. |
| `move` | `DoMove`, `DoMoveX`, `DoMoveY` | Preserve both endpoints and coordinate root. |
| size/resize | `DoResize`, `DoWidth`, `DoHeight` | Distinguish absolute size from Scale3/Scale9 geometry. |
| scale | `DoScale`, `DoScaleX`, `DoScaleY` | Do not replace an absolute legacy size with a scale unless the base asset size is recorded. |
| rotate | `DoRotate`, `DoRotateX`, `DoRotateY` | Confirm sign convention; current Z rotation is clockwise-positive in the official Basic baseline. |
| `show`/hide | `Alpha`, frame `Play`/`Stop`, or target visibility state | `Stop()` returns to the first frame but does not itself prove invisibility. |
| `color` | `SetColor(r, g, b)` where supported | Keep RGB and Alpha as separate contracts. |

### Motion groups

Legacy groups start with `:name`; each following line contains one motion with
`name`, `time`/`atime`, optional `from`/`to`, `trans`, and `repeat`. A one-line
motion may be inlined as `motion=!name=...`.

Convert each group into one or more current `animate` records with explicit
`start`, `finish`, `from`, `to`, `repeats`, `repeatType`, `delay`, and `ease` or
`custom` values. Preserve the source group and the expanded target records side
by side so repeated nodes can be compared.

- `time=A` means an animation from time zero to `A`.
- `time=A,B` means explicit start/end times.
- `time=A,+B` means start `A`, duration `B`.
- `atime` is music-linked in the legacy manual; confirm pause behavior before
  calling the current animation equivalent exact.
- `repeat=A`, `rA`, and `A,B` map respectively to repeat count, reverse mode,
  and repeat delay only when the current host contract confirms the meaning.
- Legacy `trans` is an interpolation description. Map to the nearest current
  easing/custom Bézier value and label the visual result `inferred` until
  screenshots or a host probe establish equivalence.

## Reference migration template

Use the single [conversion worksheet](#conversion-worksheet). Keep the old skin as the visual oracle and use the new host's authoritative gameplay values; do not recreate score or judgement logic from UIS.

## Failure routing and limits

- If the old entry, active include, mode, lane count, or target host cannot be
  identified, pause dependent conversion and compatibility claims; continue independent inventory and measurements.
- If a legacy built-in has no documented current equivalent, implement only a
  visual approximation with an explicit `inferred` or `unknown` record.
- If the behavior changes judgment, score, timing, Full Combo, input mapping,
  or replay semantics, route to the Gameplay checkpoint and require a focused
  probe or replay fixture.
- Do not extend this reference to Free Mode, curved/自由轨道, custom note
  simulation, or native code without the corresponding current references and
  exact-host evidence.

## Validation checklist

Use the [migration acceptance checklist](#migration-acceptance-checklist) and [proportional validation](validation.md). For converted Lua, run syntax checks. For `atime` animations, check music pause/resume. Retain actual observations separately from unverified compatibility claims.
