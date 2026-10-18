# Composer Components and Layout

Read [layout](#placement-contract) for placement work; [wire fields](#asm-wire-model-and-evidence-ledger) for ASM parsing; [conditions](#applicability-conditions-scenes) and [triggers](#triggers-priority-and-unresolved-enums) only for selection/event behavior. All numeric mappings retain their recorded target scope.

Use this reference for module creation, Composer configuration, placement, layering, media modules, masks, and script variables. Treat screenshots and target skins as observations; use `official-basic.md` for Lua API contracts.

## Skin-level settings

- Select the entry script explicitly; never infer it from nearby `.lua` files.
- Enable Free Mode only for skins implementing its required callbacks and contracts.
- Treat track tilt, track angle, scene scale, and judgment position as field settings. Read/write supported values through `Game:FieldMeta` / `Game:SetFieldMeta`; a Composer-locked field rejects script changes.
- Keep full-screen modules and play-field modules separate: the play field is transformed by scene scale/angle and is not the full screen.

## Purpose presets

Composer exposes these observed purposes: `Note`, health indicator, judgment, pause button, skip, progress, hit area, upper hidden mask, and beat line. A purpose is an engine role/preset, not a Lua module class; verify its generated behavior and mode restrictions before relying on it.

## Module and variable types

| Type | Contract/use |
|---|---|
| Image | Sprite with optional slicing, horizontal/vertical flip, color tint, and Additive blend. |
| Mask image | Single image with masking plus an observed “fill from left” option; Lua `Value` support begins at 6.2.2. Do not assume its range without a target test. |
| Solid fill | Rectangular color module; color is editable by hex value and by Lua `SetColor`. |
| Frame animation | Image sequence; control with `Play()` / `Stop()`. `Stop()` returns to frame one; set `Alpha=0` when it must disappear. |
| Text | Static or dynamic text; edit `Text`, color, layout, and font-related Composer properties. |
| Dynamic number | Choose image glyphs for `0..9` and the decimal point as the number font; configure height and inter-glyph spacing. Confirm sign, minus, empty, and rounding behavior if needed. |
| Video | Video module with width, height, removal color, removal ratio, playback speed, and loop-play toggle. Confirm codec, lifecycle, color key semantics, ratio range, and platform behavior. |
| Sound | Skin sound module with volume only. Distinguish it from Lua `Audio:Load` / `Audio:Play`; do not invent pitch, loop, or trigger properties. |
| Boolean | Composer switch read through `Module:GetBool`; missing values default to `false`. |
| Number | Runtime value is floating point through `Module:GetNumber`; the settings slider produces integers, while direct Composer editing can store decimals. |
| String | Composer text value read through `Module:GetString`; missing values default to an empty string. |

## Placement contract

- **Center point / anchor:** choose one of nine positions: four corners, four edge centers, or center. The selected point of the module coincides with `X/Y`; e.g. a bottom-right anchor makes `X/Y` the module's bottom-right coordinate and the module extends up/left.
- **Positioning point / origin:** choose the same nine positions on the entire screen. It defines the coordinate origin; e.g. screen-center origin makes `X=0,Y=0` the screen center, while top-left makes `X=0,Y=0` the screen top-left.
- **Coordinate direction:** `X` increases to the right and `Y` increases upward. This direction does not change with the selected origin, anchor, track tilt, or scene region.
- **Hit area:** choose the target track/lane number; numbering starts at `1` and increases left-to-right by track index, including tilted tracks.
- Resolve coordinates from `(origin, X/Y, anchor, Width/Height)`; never infer position from `X/Y` alone.
- For parented, play-field, Note, or group modules, confirm whether coordinates are screen-, parent-, or center-relative before applying this model.
- Official Lua contract: `Rotate` is degrees around screen Z; positive is clockwise. `RotateX/Y` are integer rotations. `Alpha` is integer `0..100`, where `0` hides and larger values are more opaque.

## Layering and draw order

- Composer exposes, from back to front: background, play area below Notes, play area above Notes, and top layer; serialized `Param` layer values are `1` background, `2` play-below-Notes, `3` play-above-Notes, `4` top (target-observed across Mutsumi and beattube, 2026-08-07).
- Play-area layers (`2`/`3`) follow scene transformations and are scaled/skewed with the scene, so their size and screen position differ from the top layer; background (`1`) and top (`4`) remain full-screen and unscaled. For overlays intended to remain screen-fixed on this target, use top layer `4`; field-attached UI can intentionally use play layers: the 437-beattube port had `beattube_bottom`/`hint` on `2` and `beattube_hp` on `3`, which scaled them with the play field; all were moved to `4` (2026-08-07).
- Within one layer, larger order values render above smaller values and may cover them; order is only compared within the same layer.

## Color, blend, flip, and masks

- Text and solid fill accept hex colors; image-like modules may support tinting through Composer or `SetColor(r,g,b)`.
- Horizontal/vertical flip changes visual orientation; re-check anchor, mask direction, sliced borders, and asymmetric assets after flipping.
- Additive blend adds source light/color to the destination: bright pixels brighten the result and dark/black pixels contribute little. Use it for glow/light effects, not opaque panels.
- For masks, verify fill direction, initial value, clipping edges, and whether `Value` is normalized or percentage-based in the exact version/module.

## Unity-style 9-slice

Use slicing for scalable frames, panels, bars, and rounded rectangles. Four border values split the sprite into nine regions: corners retain shape, edges stretch or tile along one axis, and the center fills the remaining area. Preserve sufficient border pixels and avoid resizing below the combined opposing borders. Lua 6.0+ exposes `SetSlice(left, top, right, bottom)`; confirm whether Composer/import settings use pixels and whether the target stretches or tiles each region.

## Validation checklist

Verify actual screen dimensions, origin, anchor, parent, field transform, layer, order, rotation sign, alpha endpoints, hex color, blend, flip, slicing borders, mask direction/value, and variable defaults in Composer and in-game; screenshots alone do not prove the coordinate contract.

## ASM wire model and evidence ledger

Use this section when parsing or modifying a MalodyV `info.asm`. It records current target evidence, not a substitute for a version-specific host probe.

### Evidence labels and safety boundary

- `target-observed`: directly present in the active Mutsumi ASM, Lua, or parser output.
- `editor-observed`: obtained by changing one Composer option and observing ASM or a target skin.
- `reference`: historical Composer/Lua documentation, not proof of the current host.
- `inferred`: consistent with names or instances but not a runtime contract.
- `unknown`: insufficient evidence; preserve it and do not invent a mapping.
- For read-only analysis, do not edit Steam originals or active source/ASM/metadata/DLL files. An explicit audit-and-fix request permits the authorized source patch; deployment remains separately scoped. Do not call unknown `GameAssembly` addresses. A simulator/local mock is never exact-host proof.

### Top-level protobuf fields

| Message | Field | Current interpretation | Evidence |
|---|---:|---|---|
| `SkinFile` | 1 | `Meta` | target-observed; internal fields incomplete |
| `SkinFile` | 2 | `DefaultSkinModule` disabled-ID list; raw 1-byte IDs, enable removes the ID | target/editor-observed + IL2CPP metadata parse, Windows 4K beattube, game 328450, 2026-08-07 |
| `SkinFile` | 3 | repeated `Module` | target-observed |
| `SkinFile` | 4 | `Settings` / `ComposerSetting` | target-observed; partial |
| `SkinFile` | 5 | repeated `ModuleDef` | target-observed; partial |
| `Module` | 1–13 | `Usage`, `Type`, `Meta`, `Scenes`, `Triggers`, `Param`, `Image`, `Text`, `Images`, `Number`, `Note`, `Value`, `Sound` | target-observed; enum coverage incomplete |
| `Module` | 2 | element kind: `5000` custom image, `5004` solid-color fill, `5001` text, `4900` value/bar, `7` note, `6` hold; `5002`/`5003` remain unclassified here; `4800` was later observed as a multi-frame images collection (see child mappings) | target/editor-observed, Windows 4K beattube + Mutsumi, 2026-08-07 |
| `Module` | 20 | `Animations` | target-observed; static runtime semantics incomplete |

`ModuleMeta` fields 1–7 are `Creator`, `Tags`, `Desc`, `Client`, `Time`, `Disabled`, and `HideInEditor`. In the current Mutsumi family, the Composer ID is in `Desc` (field 3); parsers may fall back to `Tags`, then a generated `__module_<index>` ID. That fallback is parser policy, not an engine contract.

### Skin field settings: Windows 4K target

A controlled edit sequence on the Windows 4K `beattube` skin (client metadata `395020`, saved by game version `328450` on 2026-08-07) mapped the nested skin `Meta` field 7 settings message. These are exact-target serialization observations; re-probe another version, platform, or mode before writing them.

| Settings field | Observed meaning | Proven values/range | Evidence and constraints |
|---:|---|---|---|
| 2 | Track slope/angle | integer `30..50`; observed `30` and `45` | target/user controlled edit. The value remains serialized while tilt is disabled, so it is a dormant configured angle, not proof that the field is visibly tilted. |
| 3 | Enable tilted track | `1 = enabled`; missing = disabled | target/user controlled edit. Do not infer other values. |
| 4 | Key/lane count | observed `4` for 4K | target-observed; mode scoped. |
| 5 | Scene scale | float; `0.4338204563` displayed as approximately `0.43` | target/user controlled edit. Preserve the float rather than rounding the serialized value. |
| 6 | Tilt permission lock | `1 = tilt use locked`; missing after the user unlocked tilt | target/user controlled edit. The exact UI wording and behavior outside this target remain version-sensitive. |
| 7 | Scene-scale lock | `1 = locked`; missing = unlocked | target/user controlled edit. Correlates with skin `Meta` field 25 and `info.json.feature` bit 0. |
| 8 | Judgment position | signed integer; missing/default `0`, observed `66`, user-confirmed UI range `-160..1500` | target/user controlled edit. Encode negative values as signed int32 semantics and verify the visible judgment line. |

`info.json.feature` is a bitmask, not a feature index. On this target, enabling feature index 0 serializes integer `1` (`1 << 0`), adds skin `Meta` field `25 = 1`, and adds settings field `7 = 1`; this locks scene scale. Keep all three representations synchronized when patching outside the game. Removing field 25 and settings field 7 while writing `feature=0` produced the unlocked representation, but the game restored them when the user locked scene scale again.

For a straight locked 4K field, the observed settings were field `4=4`, field `5≈0.4338204563`, field `6=1`, and field `7=1`, with field 3 absent. Field 2 may still store the minimum dormant slope value `30`; do not require its absence to call the visible field straight. For an enabled 45-degree tilted field, field `2=45`, field `3=1`, and field 6 was absent after tilt permission was unlocked.

Changing these skin-level settings did not alter the 15 Composer modules in the fixture. A controlled toggle on the same Windows 4K `beattube` fixture resolved where default components live: `SkinFile` field 2 is a raw list of `DefaultSkinModule` enum IDs that are **disabled**; enabling a default component removes its ID from the list. Enabling 判定 (Judge) and 打击效果 (HitLight) in the editor removed IDs `8` and `12` (`0x08`, `0x0c`) from the field-2 payload (`040b0f080c090d` -> `040b0f090d`), with no other semantic change except the `Meta` updatedTime and `info.json.updatedTime`. ID values were recovered from the `DefaultSkinModule` enum in `global-metadata.dat` (IL2CPP metadata v31): `None=0, Pause=1, HP=2, Score=3, Combo=4, Acc=5, MPRank=6, Watch=7, Judge=8, FastSlow=9, Progress=10, PlayField=11, HitLight=12, PressLight=13, ChartMeta=14, Cover=15, Catcher=16, NoteLine=17, Musicbar=18, Drum=19, TouchArea=20`. Field-2 order is disable/insertion order, not sorted by ID, and IDs such as `7` or `16..20` may be absent. `parse_skin_asm.py` does not currently emit field 2, so compare it with a full protobuf tree dump (`dump_asm_tree.py`) for byte-exact diffs. Evidence is scoped to Windows 4K beattube saved by game `328450` on 2026-08-07; re-probe another version, platform, or mode before writing these bytes.

### `ModuleParam` layout fields

| Field | Name | Current interpretation | Evidence |
|---:|---|---|---|
| 2 | `Pivot` | one of nine module anchor points, `0` top-left through `8` bottom-right | target/editor-observed |
| 3 | `Rotate` | Z rotation | target-observed; Lua sign/version contract must be checked |
| 4–5 | `X`, `Y` | positioning point coordinates | target-observed |
| 6 | `Alpha` | integer opacity, `0..100`; a missing field is displayed as `0` in the Composer, so migrated modules need an explicit `100` | target/editor-observed + raw-byte check, Windows 4K beattube, 2026-08-07 |
| 7 | `Layer` | draw layer; observed values include `1`, `2`, `4` | target-observed; the later scoped 1/2/3/4 mapping is recorded under Layering and draw order, not a universal enum |
| 8 | `Order` | signed same-layer order | target-observed; larger values are expected above smaller only where reference/host confirms |
| 9–10 | `XUnit`, `YUnit` | coordinate units: `0 Percent`, `1 Unit`, `2 PX` | target/editor-observed |
| 11–12 | `DX`, `DY` | post-position offset | target-observed |
| 13–14 | `DXUnit`, `DYUnit` | offset units: `0 Percent`, `1 Unit`, `2 PX`; `1 Unit` = 1/1080 of screen height on a 1920×1080 reference canvas (1 logical pixel at 1080p); 100% of screen height = 1080 units | target/editor-observed; Unit scale from Mutsumi `Game:Width()`=1920 at 1080p, `kpsbarbg` 300 Unit = 300 logical px (mutsumi.lua KPS `stepX=logW/physW`), ModuleParamUnit enum Px=2 vs Unit=1, 2026-08-07 |
| 15 | `AnchorNote` | Note anchoring flag | target-observed; runtime effect unknown |
| 16 | `Coexist` | coexistence flag | target-observed; runtime effect unknown |

The working placement model is `Convert(X/Y) + Convert(DX/DY)`, with the selected module pivot aligned to the result. Canvas, parent, play-field, and Note transforms remain separate unresolved contracts. Percentage points use screen coordinates `(0,100)` at top-left, `(50,50)` at center, and `(100,0)` at bottom-right; X is right-positive and Y is up-positive. Do not use raw `X/Y` as pixel coordinates without checking units and host scaling.

Element kind is chosen by `Module` field 2, not by `ModuleParam`: on the Windows 4K beattube fixture (2026-08-07), `5004` with `color` renders as a solid-color fill in the Composer and a `file` value is ignored, while `5000` with `file` is a custom image element. The migrated `beattube_bottom` was authored as `5004`; the user re-created it as a `5000` "hint" element to display `hint.png`. The same migration wrote legacy unit-scale offsets with `Percent` units: `beattube_combo/score/accuracy` carry `dy=280/335/245` that only stay on-screen as `Unit` (about 26/31/23% of a 1080-high screen each), and `beattube_bottom` had `y=100` plus `dy=110%`. Treat `Module` field 2 as the element-kind contract and re-probe other versions before changing it.

The same migration also dropped the `Alpha` field (Param field 6) from all 15 beattube modules; the Composer displays a missing `Alpha` as `0`, and writing an explicit `100` was required to make them opaque. Raw-byte presence must be checked because the semantic model defaults a missing `Alpha` to `100`.

A `5004` to `5000` conversion keeps the old `color` value, and for a `5000` image `Color` is a multiply/tint layer: `beattube_bottom` kept the solid-fill black `#000000` after being switched to `file='hint.png'`, so it rendered as a black bar in-game while the user-created `hint` element (same file/size/pivot/layer/alpha, `img.color=''`) displayed correctly. `Image` field 15 is the color string; an untinted `5000` conversion in this case needed empty color (intentional image tint remains valid), so the repair removed field 15 and re-encode the module message in original field order (byte diff observed: info.asm 1856 -> 1847, 2026-08-07).

### Child parameter mappings

- `Image`: `File`, resource, `Frames/FPS/Repeat`, `Width/Height`, `FlipX/FlipY`, `Slice`, `FileBase/FileStart`, `Fill`, `ValInt`, `Color/Hide`, width/height units, `Blend`, and `NoBreak`. Confirm `Slice` ordering; Lua `SetSlice(left, top, right, bottom)` is only a candidate mapping. Observed `Fill=0` from left and `Fill=1` from right; `Blend=0` none and `Blend=1` additive. For untinted `5000` image conversions clear inherited fill `Color`; preserve intentional tint; a residual hex from a `5004` to `5000` conversion tints the whole texture (beattube_bottom showed a black bar until field 15 was removed, 2026-08-07).
- Image `Width`/`Height` unit `0` is Percent of screen and unit `1` is Unit = 1/1080 of screen height (1920×1080 reference canvas, 1 logical pixel at 1080p) (editor-confirmed Windows 4K beattube + Mutsumi KPS `Game:Width()`=1920 evidence, 2026-08-07); the engine stretches textures to the module size and does not enforce aspect matching, so port sizes from the source skin, not from the texture. Legacy UIS `size=W%,N` ports to `Width=W` Percent + `Height=N×1.5` Unit because UIS relative units are `1 = window height/720` while MalodyV Unit is `1 = window height/1080` (source: 1173_BeatTube_Skin `_buttom` = `100%,60` → 100% x 90 Unit, `score-hp` = `30.234%,15` → 30.234% x 22.5 Unit; UIS `pos` y offsets convert the same way, e.g. `99%-6.5` → dy=-9.75 Unit). The beattube migration originally wrote plain-number heights as Percent (`100x60%`) and swapped `score-hp`'s units; the first correction only swapped units 1:1, which still under-scales the elements; the UIS×1.5 conversion fixes it.
- `Text`: `Text`, `Font`, `Color`, and `Size`; font numbering, resolution conversion, wrapping, clipping, and placeholders are not fully recovered.
- Multi-frame modules (`Type` 4800, images collection): every frame item needs `frames >= 1`, `fps >= 1`, and `FileBase` + `FileStart` to build the sequence (UIS `frame=score/0-3` -> `FileBase='score-'`, `FileStart=0..3`). A 4800 item carrying only `File` + `Width/Height` with `frames=0` renders a white placeholder box; `beattube_judge` had `250% x 250%` Percent items with no frames and showed a giant white square until its images were replaced with the working `judge` element config (frames=1/fps=1, FileBase='score-', FileStart=0..3, item0 100x100 Unit, later items 0=inherit), 2026-08-07. Put fixed-size judge sprites in Unit, not Percent.
- `Note` modules (`Type` 7 tap / `Type` 6 hold, field 11): the images collection is a fixed-size slot array, the recorded target skins use 2 items for `Type 7` and 5 for `Type 6`; unused states keep an empty item message (0 bytes) in place, never delete the item because removing one shifts later textures to wrong slots. Cross-skin evidence (Mutsumi, SKY-edited-cynosure_key, Elaina_4K_Ultra, Quaver, 2026-08-07) shows the extra slots empty by default. 437 `tex4=noteto.png` is the multi-press/overlap state texture and has no MalodyV counterpart; on beattube that slot was emptied (item replaced with 0 bytes, info.asm 1849 -> 1749, file `noteto.png` deleted) while `noteh` stays at slot 4, 2026-08-07.
- `Number`: `File`, `Text`, `Height`, `Padding`, `FileBase`, `FileStart`, and `ValInt`; sign, decimal, leading zero, rounding, maximum digits, padding units, and `Text` versus `ValInt` precedence are unresolved.
- `Value`: `ValF`, `ValS`, `ValB`, `ValMin`, and `ValMax`. High-confidence type mappings are `4700` number/float, `4701` string, and `4702` bool. Missing bool/string values default to `false`/empty string in the observed Lua contract.
- `Sound`: `File`, `Loop`, `Volume`, and `HitSound`; playback timing and exact runtime behavior remain unverified, and it must not be conflated with Lua `Audio:Load`/`Audio:Play`.
### Static animation runtime contracts

- Static `Animations` serialize `Type`, start/end times, up to two from/to values, repeat, repeat type, per-repeat delay, ease, and `Nid`. The exact-host mapping for `Type` is `1 MoveX`, `2 MoveY`, `3 Move(X,Y)`, `4 Width`, `5 Height`, `6 Size(W,H)`, `7 ScaleX`, `8 ScaleY`, `9 Scale(X,Y)`, and `10 Alpha`.
- Animation time values are milliseconds. Music start is `0`; the interval before music starts is negative. On the tested host, `repeat=0` runs once. `Delay` is the delay for each repeat. Positive repeat counts and repeat-type boundary behavior remain unproved.
- Ease values are `0 linear`, `1 ease-in`, and `2 ease-out`. A 20-to-100 Alpha animation over 10,000 ms sampled near 5,000 ms produced approximately `60`, `41`, and `79`, respectively. The final-value probe reached the configured targets for Types `1..10`.
- For overlapping animations on the same property, the smaller `Nid` won in the reversed-order exact-host probe (`Nid=100` versus `Nid=900`). Do not extend this result to equal IDs, cross-property conflicts, or other versions without a probe.
- A Trigger attached to a static Animation does not make that animation event-relative on the tested host: the static animation follows the music timeline, while a later event can reset the triggered module without replaying the animation. Use Lua/frame-animation ownership for event feedback unless the target proves a different path.

### Animation/Lua ownership and Retry

- Treat each mutable Composer property as having one owner per frame. A repeated Lua write from `Update()` held an ASM-animated Alpha at the Lua value in the observed target; a one-shot write from `OnInput()` or `OnHit()` survived the immediate next sample but required later writes to remain visible when another owner could update the property.
- `OnRetry()` is delivered during the negative pre-music interval in the tested 4K Play target, and Retry does not rerun `Init()`. Retry reset/re-evaluated the tested ASM animation state, while Lua variables and ordinary Lua-written properties persisted unless `OnRetry()` explicitly clears or restores them. ASM-animated properties may be reasserted by the engine timeline after Retry.
- A cached Module reference compared equal to a fresh `Module:Find()` reference during Trigger input and after Retry in the tested target. Rebind after lifecycle boundaries when supporting another version, mode, or module-construction path.
- One observed event cluster was `OnInput -> Update -> OnHit -> Update`; treat this as target evidence, not a universal callback order. Build callback-dependent logic so it remains correct if events are separated across frames.
### Applicability conditions (`Scenes`)

The editor label is “applicability condition”; parser output `applicability_conditions` must not be treated as a Unity scene list. Each condition has `Source`, `Flag`, signed `ValInt`, `ValDouble`, and `ValString`. Comparison flags are `0 =`, `1 !=`, `2 >`, `3 <`, and `4 occurred/event`; event semantics are not established.

| Source | Editor parameter | Value | Evidence/status |
|---:|---|---|---|
| 1 | Note horizontal position / lane X | int | editor/runtime probe; Note-instance scoped |
| 2 | — | — | reserved/unknown |
| 3 | Screen width | int | editor/runtime probe; `>1` showed in test, `<1` hid |
| 4 | Screen height | int | editor-observed mapping; runtime edge matrix incomplete |
| 5 | Aspect ratio width/height | double | editor-observed mapping; runtime edge matrix incomplete |
| 6 | Runtime platform | int | editor/runtime probe; Windows target is `1`, `0` hid |
| 7 | Note type | int | editor-observed mapping; instance semantics not fully probed |
| 8 | Track tilt | int | editor-observed mapping; runtime semantics incomplete |
| 9 | Scene scale | double | editor-observed mapping; runtime semantics incomplete |
| 10 | — | — | reserved/unknown |
| 11 | — | — | reserved/unknown |
| 12 | Player | int | editor-observed mapping; runtime semantics incomplete |
| 13 | Mod type | int | editor/cross-skin observed values `16`, `32`, `256`; names/bitmask semantics unresolved |
| 14 | Judgment difficulty | int | editor-observed mapping; runtime semantics incomplete |
| 15 | Beat subdivision | int | editor-observed mapping; runtime semantics incomplete |

Runtime applicability order proven by the disposable `skin/test` probe: (1) evaluate conditions that the target can establish and remove inactive modules; (2) for ordinary `Module:Find(id)`, the later remaining active module wins among duplicate IDs. Two active duplicate probes returned the later module; making it inactive returned the earlier module. This is a target/editor observation for the tested version, not a universal engine promise.

`Source=1` has two distinct scopes. Ordinary Text UI modules with `Note X=1` and `Note X=4` both stayed visible continuously, so Source 1 is not a global static filter for ordinary UI in that fixture. Actual Note modules (`Usage=1`, `Type=7`) filtered per Note instance: red `Note X=1` art appeared only on lane 1 and blue `Note X=4` art only on lane 4. Same-ID Note templates coexisted during Note construction. Do not infer Lua `Module:Find` behavior inside or outside Note construction from this result.

The simulator may filter only conditions backed by an explicit scenario value. Unknown Note-X, event, multi-condition, or other unresolved inputs remain active and must emit an uncertainty diagnostic; simulator success is not exact-host proof.

### Triggers, priority, and unresolved enums

`Triggers` are separate from `Scenes` even though their wire shape is similar. Do not reuse the applicability Source table for triggers. An exact-host Windows 4K Play probe established these trigger mappings for that target scope:

| Source | Trigger parameter | Proven values | Evidence |
|---:|---|---|---|
| 7 | Input state | `1 = pressed`, `2 = move`, `3 = released` | probe-observed |
| 8 | Hit judgment | `0 = blank`, `1 = Best`, `2 = Cool`, `3 = Good`, `4 = Miss`, `5 = Hold`, `6 = slide` | probe-observed |
| 9 | Early timing | value `1`; both `occur` and equality forms were observed | probe-observed |
| 10 | Late timing | value `1`; both `occur` and equality forms were observed | probe-observed |
| 15 | Note operation | `1 = hit`, `2 = hold`, `3 = valid slide`; equality and inequality forms were observed | probe-observed |

Observed condition forms additionally include `T7 != 1` (input is not pressed), `T9 OCCUR 1` / `T9 = 1` (early), `T10 OCCUR 1` / `T10 = 1` (late), and `T15 != 2` (note operation is not hold). These are target-observed forms, not a complete enumeration of every Source/comparison combination.

- Multiple Trigger records use AND semantics on the tested host. A Best hit activated the combined `T8=Best + T15=hit` module, while a Cool hit activated only the single `T15=hit` control and not the combined or single-Best controls. Inactive Trigger modules read Alpha `0`; active controls exposed their serialized Alpha in the probe. Treat this as a condition/visibility observation, not a guarantee that every module type exposes the same property state.
- The tested 4Key mode supplied no horizontal move for the Hold case, so `T7=move + T15=hold` could not be used to characterize Hold movement. Probe the mode-specific input path before relying on Sources 7, 8, or 15 outside this scope.
- Trigger activation samples were event-dependent (the probe observed about 85 ms for press and 21 ms for Best); do not treat those durations as universal. Trigger duration, Retry reset of Trigger activation state, and full ordering relative to Lua callbacks remain version/mode-sensitive.

For module selection, use this evidence-backed priority model: applicability filtering precedes duplicate-ID indexing; later active duplicates replace earlier entries for ordinary `Module:Find`. Do not extend that priority rule to Note-instance template construction, layer/order rendering, animation conflicts, or triggers without a dedicated probe. Preserve `Usage` values `1/2/7/10/98/99` and `Type` values `6/7/4500/4800/4900/5000–5004` as partially mapped rather than inventing complete enums.

### Cross-skin evidence and probe matrix

Read-only scanning of 17 other Steam skins found 188 Source-1 conditions, 16 Source-6 conditions, and four Source-13 conditions; no Source 2, 10, or 11 samples appeared. Duplicate IDs appeared in six skins: `analyzer`, `b64a49bfff30634f40a8f9c08edadcbd`, `Mango233_4K_Skin`, `Mutsumi`, `Mutsumi_v2.1`, and `Mutsumi_v2_obf`. Use this as prevalence evidence, not semantic proof.

When probing, change one ASM/Composer variable at a time, keep a timestamped backup outside the Steam source, use a visible unique label or color, test both boundary directions, and restore the file after each run. Required distinctions are: ordinary UI versus Note module, inactive versus active duplicate, platform `0/1`, width just below/above threshold, and known versus unresolved Source values. Record exact target version, mode, active skin path, visible result, and rollback state.

### ASM response and acceptance contract

For claims about ASM semantics, include the relevant facts concisely: (1) the parsed field or condition, (2) its module scope, (3) an evidence label, (4) the exact result that is proved, and (5) the next validation step or explicit `unknown`. For a requested edit or test skin, also state the single changed variable, source and deployment paths, timestamped backup path, boundary cases, expected visible result, and rollback action.

Use these scope-specific acceptance checks:

| Request | Required conclusion | Do not conclude |
|---|---|---|
| `Source=1` on ordinary UI | whether the chosen UI fixture is statically active | that Note templates use the same activation path |
| `Source=1` on a Note module | lane-specific art is selected per Note instance | ordinary Lua duplicate-ID lookup behavior |
| ordinary duplicate `Module:Find` | conditions filter first; the later active duplicate wins | Note-template overwrite, draw order, or trigger precedence |
| `Source=3` or `Source=6` | tested comparison boundary or platform value on the exact host | untested height, aspect, mod, or event semantics |
| unknown Source/Trigger/animation field | the parser’s raw value and missing evidence | a guessed enum label or runtime behavior |

Report result-changing unknowns and the host observation needed to resolve them. A narrow field edit needs only its affected evidence, not a repeated full ASM ledger.
