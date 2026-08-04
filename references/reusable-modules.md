# Reusable MalodyV Lua Modules

Use this catalog after identifying the target skin's actual entry, mode, callbacks, Composer nodes, settings, and data sources. It converts common requests into a small implementation contract and a Lua 5.3 skeleton. It does not grant permission to assume a node ID, a callback API, an event enum, a coordinate system, or a Native module exists.

## Catalog rules

For every selected module, record:

```text
target callback path
authoritative input source and unit
required versus optional Composer nodes
state created at Init and state reset at Retry/reload
one owner for each written node property
cached display/animation values
in-game acceptance condition
```

Use generic IDs such as `<score_text_id>` only as planning placeholders. Before writing production Lua, replace each one with a target-proven Composer ID or a parameter read from the target's existing configuration. Do not copy IDs, Chinese Composer labels, asset names, or settings from a reference skin into an unrelated skin.

When a request combines a display and an animation, implement in this order:

1. Confirm and normalize the data source.
2. Define display formatting, cache, and retry/reset behavior.
3. Bind the target node and confirm its base layout/anchor.
4. Add explicit animation state and one final property-apply path.
5. Validate the display without the animation, then validate the animation.

## 1. Lifecycle dispatcher

**Use for:** a multi-feature skin whose public `Init`, `Update`, `OnHit`, `OnInput`, and `OnRetry` functions should remain thin.

**Do not use for:** a one-off single-file edit where a dispatcher would only hide a few lines of code.

**Contract:** feature handlers receive no implied host objects. Each handler is responsible for the host APIs it has verified. `update`, `on_hit`, `on_input`, and `on_retry` must not run before successful initialization.

**State/reset:** store only the handler table and initialized flag. Reset behavior belongs to individual features, dispatched through `on_retry`.

```lua
local runtime = { handlers = {}, initialized = false }

local function call(name, ...)
    local handler = runtime.handlers[name]
    if handler then
        return handler(...)
    end
end

function runtime.configure(handlers)
    runtime.handlers = handlers or {}
end

function runtime.init()
    call("init")
    runtime.initialized = true
end

function runtime.update(...)
    if runtime.initialized then
        call("update", ...)
    end
end

function runtime.on_hit(...)
    if runtime.initialized then
        call("on_hit", ...)
    end
end

function runtime.on_input(...)
    if runtime.initialized then
        call("on_input", ...)
    end
end

function runtime.on_retry(...)
    if runtime.initialized then
        call("on_retry", ...)
    end
end
```

**Acceptance:** a missing optional feature does not block unrelated initialized features; no event handler runs before `init` completes.

## 2. Configuration and locale adapter

**Use for:** Composer settings, optional toggles, numeric parameters, and localized UI strings.

**Target discovery:** verify the existing setting key, expected type, default behavior, and language selection mechanism. Settings keys are stable contracts, not display text to be translated or renamed.

**State/reset:** cache static settings at `Init`; re-read only when the host supports settings changes during play or the target already does so.

```lua
local config = {}

function config.read_bool(key, default)
    local ok, value = pcall(function()
        return Module:GetBool(key)
    end)
    if ok and value ~= nil then
        return value == true
    end
    return default == true
end

function config.read_number(key, default)
    local ok, value = pcall(function()
        return Module:GetNumber(key)
    end)
    value = ok and tonumber(value) or nil
    return value == nil and default or value
end

function config.read_string(key, default)
    local ok, value = pcall(function()
        return Module:GetString(key)
    end)
    if ok and value ~= nil then
        return tostring(value)
    end
    return default
end
```

**Acceptance:** missing optional settings use documented deterministic defaults; missing required settings fail with the key and feature name.

## 3. Node binding, visibility, and text cache

**Use for:** any feature that finds Composer nodes, toggles visibility, or displays changing text.

**Target discovery:** confirm node ID, node type, parent/anchor, initial value, and whether the node is required. Cache references in `Init`; never call `Module:Find` every frame unless the target explicitly creates/destroys nodes dynamically.

**Property ownership:** the feature owns only the properties it changes. If another feature owns `Alpha` or `Text`, combine outputs in a single adapter rather than both writing independently.

```lua
local ui = { node = nil, last_text = nil, enabled = true }

local function bind_required(id)
    local node = Module:Find(id)
    assert(node, "required Composer node is missing: " .. tostring(id))
    return node
end

local function bind_optional(id)
    return Module:Find(id)
end

local function set_visible(node, visible)
    if node then
        node.Alpha = visible and 100 or 0
    end
end

local function set_text_cached(node, text)
    text = tostring(text)
    if node and ui.last_text ~= text then
        ui.last_text = text
        node.Text = text
    end
end

local function reset_text_cache()
    ui.last_text = nil
end
```

**Acceptance:** absent optional nodes disable only the local feature; retry clears cache so the initial post-retry value is written.

## 4. Numeric display adapter

**Use for:** score, accuracy, combo, judgement counts, KPS, progress, time, BPM, and any numeric text.

**Target discovery:** verify source ownership, unit, range, rounding, update callback, and whether the value is engine-provided or custom. Do not recreate a score/accuracy calculation when a target-proven authoritative value already exists.

For score, judgement-count, difficulty-label, result-grade, or Full Combo UI, read `player-gameplay-rules.md` for player-facing semantics. Keep live source values, final result values, and custom estimates visibly distinct.

**State/reset:** store numeric source state separately from formatted text; reset counters, last formatted value, and time windows on retry/reload.

```lua
local display = { last_key = nil, node = nil }

local function percent_text(value, decimals)
    local number = tonumber(value) or 0
    return string.format("%." .. tostring(decimals) .. "f%%", number * 100)
end

local function write_number(key, text)
    if display.node and display.last_key ~= key then
        display.last_key = key
        display.node.Text = text
    end
end

local function update_accuracy(raw_accuracy)
    local value = tonumber(raw_accuracy) or 0
    local key = math.floor(value * 10000 + 0.5)
    write_number(key, percent_text(value, 2))
end

local function reset_display()
    display.last_key = nil
end
```

**Acceptance:** first event, zero values, misses, empty metadata, and retry all display intentionally; the same visible text is not written every frame.

## 5. Multi-instance animation controller

**Use for:** movement, scale, alpha, pulse, key effects, repeated judgement elements, cloned bars, and lane-specific feedback.

**Target discovery:** confirm coordinate root, anchor, parent transforms, base `X/Y`, base scale/size, alpha unit, time unit, and whether the node supports each intended property.

**State/reset:** keep immutable base layout and per-instance transient state. On retrigger, explicitly choose restart, replace, blend, or ignore. Reset every active instance on retry, chart reload, and time rollback.

```lua
local animation = {}

local function clamp(value, low, high)
    if value < low then return low end
    if value > high then return high end
    return value
end

function animation.new(node)
    return {
        node = node,
        base_x = node and node.X or 0,
        base_y = node and node.Y or 0,
        base_alpha = node and node.Alpha or 100,
        start_ms = 0,
        duration_ms = 0,
        offset_x = 0,
        offset_y = 0,
        active = false,
    }
end

function animation.trigger(state, now_ms, duration_ms, offset_x, offset_y)
    state.start_ms = now_ms
    state.duration_ms = math.max(1, duration_ms)
    state.offset_x = offset_x or 0
    state.offset_y = offset_y or 0
    state.active = true
end

function animation.update(state, now_ms)
    if not state.node or not state.active then return false end
    local progress = clamp((now_ms - state.start_ms) / state.duration_ms, 0, 1)
    local remaining = 1 - progress
    state.node.X = state.base_x + state.offset_x * remaining
    state.node.Y = state.base_y + state.offset_y * remaining
    if progress >= 1 then
        state.active = false
    end
    return state.active
end

function animation.reset(state)
    if state.node then
        state.node.X = state.base_x
        state.node.Y = state.base_y
        state.node.Alpha = state.base_alpha
    end
    state.active = false
end
```

**Acceptance:** repeated triggers do not drift; each clone animates independently; final position is always derived from base layout, never the previous rendered frame.

## 6. Clone-based key overlay

**Use for:** per-lane key effects, hit shadows, press counters, lane labels, and other repeated elements that must scale with chart key count.

**Target discovery:** verify chart key-count source, template nodes, clone support, clone naming rules, parent/anchor behavior, and the target skin's lane order. Never assume a 4K/7K/8K layout or template exists.

**State/reset:** each lane owns its node references, press count, animation state, and cached display. Clone once at `Init`; do not clone inside `Update` or an input callback.

```lua
local overlay = { lanes = {}, count = 0 }

local function add_lane(index, template, make_id)
    local node = index == 1 and template or Module:Clone(template, make_id(index))
    overlay.lanes[index] = {
        node = node,
        presses = 0,
        animation = node and { base_alpha = node.Alpha, until_ms = 0 } or nil,
    }
end

local function register_press(lane_index, now_ms, duration_ms)
    local lane = overlay.lanes[lane_index]
    if not lane then return end
    lane.presses = lane.presses + 1
    if lane.animation then
        lane.animation.until_ms = now_ms + duration_ms
    end
end

local function update_lanes(now_ms)
    for index = 1, overlay.count do
        local lane = overlay.lanes[index]
        if lane and lane.node and lane.animation then
            local active = now_ms < lane.animation.until_ms
            lane.node.Alpha = active and 100 or lane.animation.base_alpha
        end
    end
end
```

**Acceptance:** key count determines instance count; a single input only affects its mapped lane; retry clears counts and inactive effects.

## 7. KPS history and graph renderer

**Use for:** key-per-second text, rolling input windows, line graphs, histograms, or recent-input visualizations.

**Target discovery:** verify input callback/event type, timer source, graph template node, clone behavior, graph coordinate root, and whether inputs should include all events or only successful presses.

**State/reset:** maintain bounded timestamps/history, current input bucket, cached scale, and clone references. Reset the entire history on retry and time rollback.

```lua
local kps = { times = {}, head = 1, window_ms = 1000 }

local function prune_inputs(now_ms)
    while kps.head <= #kps.times and kps.times[kps.head] < now_ms - kps.window_ms do
        kps.head = kps.head + 1
    end
end

local function record_input(now_ms)
    kps.times[#kps.times + 1] = now_ms
    prune_inputs(now_ms)
end

local function current_kps(now_ms)
    prune_inputs(now_ms)
    return #kps.times - kps.head + 1
end

local function reset_kps()
    kps.times = {}
    kps.head = 1
end
```

**Acceptance:** the input list stays bounded, KPS uses one documented time window, and renderer writes only affected bars/points when values change.

## 8. Clock and progress module

**Use for:** elapsed time, remaining time, song progress, countdowns, and time-driven progress bars.

**Target discovery:** verify time source, total duration source, progress direction, alpha convention, base bar geometry, and behavior when duration is absent or zero.

**State/reset:** cache text/last progress bucket; reset cached values and base geometry when retry/reload occurs.

```lua
local clock = { text = nil, bar = nil, last_second = nil, base_width = 0 }

local function update_clock(now_ms, duration_ms)
    local total = tonumber(duration_ms) or 0
    if total <= 0 then return end
    local elapsed = math.max(0, now_ms)
    local second = math.floor(elapsed / 1000)
    if clock.text and clock.last_second ~= second then
        clock.last_second = second
        clock.text.Text = string.format("%d:%02d", math.floor(second / 60), second % 60)
    end
    if clock.bar then
        local progress = math.max(0, math.min(1, elapsed / total))
        clock.bar.Width = clock.base_width * progress
    end
end
```

**Acceptance:** zero/unknown duration is safe; progress clamps to the intended range; retry restores the target skin's base bar state.

## 9. Metric adapters: MM/S, dual accuracy, and Etterna accuracy

**Use for:** displaying a target-provided or Native-provided metric without mixing its algorithm with Composer output.

**Target discovery:** determine whether the target already computes the metric, which result/judgement mapping it uses, the metric's unit/range, and any required settings. For dual accuracy, verify both judgement profiles. For Etterna-style accuracy, verify level/profile semantics.

**State/reset:** cache last rounded display keys and reset them at retry. Metric algorithms and UI rendering must have one clear direction: source adapter -> normalized number -> formatter -> node.

```lua
local metric = { primary = nil, secondary = nil, last_primary = nil, last_secondary = nil }

local function write_metric(node, value, decimals, last_key_name)
    local number = tonumber(value) or 0
    local key = math.floor(number * (10 ^ decimals) + 0.5)
    if metric[last_key_name] ~= key and node then
        metric[last_key_name] = key
        node.Text = string.format("%." .. tostring(decimals) .. "f%%", number)
    end
end

local function reset_metrics()
    metric.last_primary = nil
    metric.last_secondary = nil
end
```

**Acceptance:** a disabled metric hides only its own nodes; each profile displays the expected unit and is reset independently.

## 10. Judgement, offset, score, and combo visual adapter

**Use for:** alternate judgement displays, offset bars, score formatting, combo smoothing, judgement counters, and timed hit feedback.

**Target discovery:** confirm event extraction (`JudgeResult`, offset, note/lane fields), enum mapping, offset sign/unit, score source, combo source, UI node family, and judgement visibility rules. This module must not silently reinterpret a host enum.

**State/reset:** keep last judgement timestamp, active node, score/accuracy/combo display cache, timed effect state, and all counters. Clear them on retry and time rollback.

```lua
local judge = { last_result = nil, last_time = 0, active_node = nil }

local function normalize_hit(event)
    if not event then return nil end
    return {
        result = tonumber(event:JudgeResult()),
        offset_ms = tonumber(event:Offset()),
        note_type = tonumber(event:NoteType()),
        lane = tonumber(event:HitX()),
    }
end

local function show_judgement(result, now_ms, node_by_result)
    local node = node_by_result[result]
    if not node then return end
    judge.last_result = result
    judge.last_time = now_ms
    judge.active_node = node
    node.Alpha = 100
end

local function reset_judgement(nodes)
    judge.last_result = nil
    judge.last_time = 0
    judge.active_node = nil
    for _, node in pairs(nodes) do
        if node then node.Alpha = 0 end
    end
end
```

**Acceptance:** raw host events are normalized once; all result mapping is explicit; a missing judgement node does not corrupt score/combo state.

## 11. Native bridge and UI snapshot

**Use for:** a target skin that already ships a tested Native module, or a deliberate performance migration with ABI/probe/replay coverage.

**Do not use for:** ordinary text, layout, animation, or small event logic. Never require a Native module merely because a reference skin uses one.

**Contract:** Lua owns MalodyV APIs, Composer nodes, asset/animation calls, event extraction, and UI writes. Native receives verified scalar values only. Document all input order, output order, units, reset semantics, and ABI version.

```lua
local function load_optional_native(module_name)
    local ok, native_or_error = pcall(require, module_name)
    if not ok then
        return nil, tostring(native_or_error)
    end
    return native_or_error, nil
end

local function call_native_update(native, time_ms)
    if native and native.update then
        native.update(time_ms)
    end
end

local function reset_native(native)
    if native and native.reset then
        native.reset()
    end
end
```

**Acceptance:** standalone `require`, ABI/self-test, exact-host load, lifecycle probe, and replay all pass before production deployment. When Native is optional, the Lua fallback is explicit and never displays stale values.

## 12. Probe and replay harness

**Use for:** unverified callbacks, event shapes, loader behavior, timing, reset, Native integration, or a gameplay algorithm migration.

**Contract:** probes are isolated from the production entry path. Record the callback, input, timestamp/unit, result, and expected observation. Replays use scalar normalized events and compare every event for zero-difference work.

```lua
local probe = { init = 0, update = 0, input = 0, hit = 0, retry = 0 }

local function mark(name)
    probe[name] = (probe[name] or 0) + 1
end

function Init()
    mark("init")
end

function Update()
    mark("update")
end

function OnInput()
    mark("input")
end

function OnHit()
    mark("hit")
end

function OnRetry()
    mark("retry")
end
```

**Acceptance:** host logs or a dedicated optional status node prove callback entry; probes do not modify or depend on the production feature graph.

## Selection guide

```text
"show score/accuracy/combo"          -> Numeric display + Node binding
"add KPS or input graph"             -> KPS history + Node binding + Clone/animation as needed
"make a key effect"                  -> Clone-based key overlay + Multi-instance animation
"add a fading/sliding UI effect"     -> Multi-instance animation + Node binding
"add timer/progress"                 -> Clock/progress + Numeric display
"show alternate judgement/offset"    -> Judgement visual adapter + Numeric display
"show MM/S, dual ACC, Ett accuracy"  -> Metric adapter; confirm algorithm/source first
"optimize a proven hot path"         -> Native bridge only after profiling and probes
"API/callback does not behave"       -> Probe and replay harness before changing production logic
```

Read `reference-module-provenance.md` when a module needs implementation provenance, source-function names, project-specific dependencies, or a known working lifecycle ordering.
