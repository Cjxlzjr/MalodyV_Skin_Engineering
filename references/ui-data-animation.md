# UI Data and Animation Contracts

Use the data, animation, coordinates, speed, or configuration section that the change touches. Validate affected properties and lifecycle paths; this is not a requirement to redesign or test every HUD feature.

Use this reference for player-visible numeric/text displays, layout changes, movement, scale, alpha, visibility, clones, and localized formatting.

## Contents

- [Data Presentation](#data-presentation)
- [Animation Ownership](#animation-ownership)
- [Coordinates and Anchors](#coordinates-and-anchors)
- [Play-Speed Handling](#play-speed-handling)
- [Configuration and Localization](#configuration-and-localization)
- [Focused Acceptance](#focused-acceptance)

## Data Presentation

For score, accuracy, combo, KPS, progress, judgment counters, and similar displays:

1. Confirm the authoritative source, unit, range, callback, and whether the value is live, final, local, or engine-provided.
2. Keep normalized numeric state separate from formatted display text. Multiple nodes may share a value, but they must not calculate independent variants.
3. Update event-driven values from the owning event. Use `Update()` for time-continuous values or visible interpolation, not for repeated chart scans or unchanged string formatting.
4. Define reset behavior for retry, chart reload, mode transition, and detected time rollback.
5. Specify precision, percentage conversion, signs, leading zeros, locale, and fallback text.
6. Select relevant boundaries: `nil`, zero notes/duration, empty metadata, first event, miss, negative offset, long text, or unusually large values.
7. Cache the last visible value and write a Composer property only when its rendered result changes.

Player-facing rules can clarify labels and acceptance, but they do not replace target-proven score, grade, accuracy, judgment, hold, or Full Combo contracts. Prefer engine-provided final values over reimplementing scoring from documentation.

## Animation Ownership

For movement, scaling, fades, pulses, reveals, and repeated elements:

1. Store immutable resting position, scale, alpha, and visibility separately from transient offsets or multipliers.
2. Assign one owner for each written property. If layout and feedback both influence `X`, `Y`, scale, alpha, text, or visibility, combine their inputs in one final apply function.
3. Record start time, duration, initial/target state, active flag, retrigger policy, and per-instance identity.
4. Choose restart, replace, blend, ignore, or queue behavior explicitly on retrigger.
5. Use one documented time unit, clamp interpolation progress, and define reset or rebase on time rollback.
6. Keep cloned instances independent unless synchronization is intentional.
7. Restore resting state deterministically when the effect ends or lifecycle reset occurs.

Compute from the base state rather than the previous rendered frame:

```lua
local rendered_x = base_x + offset_x
local rendered_y = base_y + offset_y
local rendered_scale = base_scale * pulse_scale
```

Common symptoms identify the broken contract:

```text
wrong start position   -> coordinate root, parent transform, or anchor
drift after retrigger  -> rendered output reused as base state
flicker or snap-back   -> multiple property owners
all clones animate     -> shared node reference or shared mutable state
retry leaves stale UI  -> incomplete reset/resting-state restoration
frame stutter          -> repeated lookup, allocation, formatting, or writes
plausible wrong value  -> source, unit, rounding, event, or reset mismatch
```

## Coordinates and Anchors

Treat positioning as an engine contract:

1. Confirm whether `X/Y` are screen-, parent-, panel-, field-, or center-relative.
2. Resolve placement from origin, `X/Y`, anchor, parent transforms, and actual width/height; never infer position from `X/Y` alone.
3. Read real active dimensions before layout math. A screenshot or source image size does not prove rendered dimensions.
4. Keep one coordinate system per layout and make every conversion explicit.
5. For multiline text, treat the anchor as belonging to the whole rendered block unless the target proves another rule.
6. Anchor the row or element that must remain fixed, then derive neighboring positions from it.
7. For an unknown coordinate mapping, probe one variable at a time; once verified, related fields may be patched together with before/after evidence.
8. Validate final coordinates numerically and visually in Composer or the host.

Example panel-relative conversion:

```lua
local x = panel_x + panel_width * relative_x
local y = panel_y + panel_height * relative_y
```

Example row layout derived from a fixed bottom row:

```lua
local step = -(base_step + extra_pixels / panel_height)
local start_y = fixed_row_y - step * (row_count - 1)
local row_y = start_y + step * (index - 1)
```

Verify the target's Y direction and anchor before copying the signs.

## Play-Speed Handling

When supported in the exact callback, `Game:PlayMeta("speed")` is the runtime source for the effective play-speed multiplier.

- Normalize with `tonumber`; accept only positive values such as `0.8`, `1.0`, `1.2`, or `1.5`.
- Read it in the callback that uses it; availability elsewhere does not prove callback-wide support.
- The example below is for optional visual timing only: use a neutral `1.0` fallback when that behavior is acceptable. Required judgement/offset normalization must report unavailable speed instead of silently assuming `1.0`.
- Do not also parse speed mods or apply a second turbo correction after a valid runtime multiplier.

```lua
local function read_play_speed()
    local ok, value = pcall(function()
        return Game:PlayMeta("speed")
    end)
    value = ok and tonumber(value) or nil
    return value and value > 0 and value < math.huge and value or 1.0
end
```

## Configuration and Localization

Keep these namespaces separate:

```text
Composer configuration keys -> stable keys used by Module:Get*
Composer UI IDs             -> stable identifiers used by Module:Find
localized text keys         -> display-only labels and format strings
```

- Preserve setting keys and UI IDs across languages.
- Centralize keys and IDs in one adapter; expose every reachable getter explicitly.
- Distinguish missing, invalid, false, empty, and defaulted results when behavior depends on the difference.
- Treat each format string as fixed or parameterized and test representative values for every supported locale.
- Keep numeric calculation outside the locale layer; format at the UI boundary and cache by normalized value plus locale.
- Use a deterministic fallback locale when an optional language setting is absent, while surfacing required configuration failures.

Prefer the target's compatible configuration adapter. The [catalog example](reusable-modules.md) is optional and does not detect missing keys when host getters return false/zero defaults.

## Focused Acceptance

For the affected UI behavior, record applicable fields; animation details are unnecessary for a static text fix:

```text
authoritative value and unit
target node and property owner
resting layout and anchor
update or trigger callback
format and locale behavior
animation start/end/duration/retrigger policy
retry/reload/time-rollback behavior
missing-node or missing-setting behavior
exact in-game mode, action, and expected visible result
```

Lua parsing proves none of the visual contracts. When host access is unavailable, report static/focused checks separately and give the exact manual verification procedure.
