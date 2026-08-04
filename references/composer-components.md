# Composer Components and Layout

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

- Composer exposes, from back to front: background, play area below Notes, play area above Notes, and top layer.
- Play-area layers follow scene transformations; background/top remain full-screen unless the target demonstrates otherwise.
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
