# Implementation Safety

Read the sections for the changed boundary or state owner. Pure documentation edits need no chart/runtime inventory. Required gameplay data must fail clearly when invalid; optional presentation may use a documented fallback. Preserve raw event/report evidence separately from resettable derived state.

## Engine boundaries

Keep engine calls at adapter boundaries. Pure modules accept plain Lua data and return stable Lua results. At each boundary:

- Normalize with explicit names such as `time_ms`, `offset_ms`, `duration_ms`, `rate`, and `judge_scale`.
- Validate `nil`, empty, malformed, negative-duration, out-of-range, and non-finite values.
- Preserve original engine units until an explicit conversion point.
- Separate raw engine values from processed display/analysis values.
- Never reuse a display label as a numeric result.

## Notes and charts

Before parsing/analyzing, confirm:

- Note count source/index range and lane/key count/index base.
- Start, end, duration, type, group, width, and identifier fields.
- Chord grouping and duplicate-note behavior.
- Timing-point and BPM units.
- Effects of rate, mod, judge scale, and time transforms.

Use a normalized internal note shape only after documenting conversion. Preserve identity when deduplicating. If no identifier exists, use a deterministic fallback and state collision risks.

## Judgments and input

- Treat judgment values as engine contracts; confirm enum mapping before score/statistics changes.
- Process each judgment exactly once even when native hit and manual input callbacks overlap.
- Keep Miss handling explicit; never silently discard Miss samples or replace engine offsets without documenting the rule.
- Separate manual input-speed metrics from AutoPlay/generated judgments.
- Track long-note head, body, release, and break independently when exposed separately.

## Architecture

Prefer small modules with explicit APIs:

```lua
local M = {}

function M.update(state, input)
    -- Keep this function independent of engine globals.
end

return M
```

Recommended layers:

1. Engine adapter - callbacks, Composer access, mode checks, lifecycle.
2. Normalization - chart, input, judgment, timing, modifier conversion.
3. Pure calculation - difficulty, statistics, geometry, state transitions.
4. Presentation - text, graphs, animation, particles, optional diagnostics.
5. Tests - minimal mocks, fixture charts, regression cases.

Separate parsing, calculation, state mutation, and UI formatting where practical. Return stable documented result tables. Distinguish missing, invalid, fallback, and authoritative results.

## Performance and cache safety

- Cache immutable modules and chart-derived calculations.
- Avoid full-chart scans, repeated large string concatenation, allocations, `Module:Find`, and cloning in per-frame/per-note callbacks.
- Prefer dirty flags, incremental cursors, ring buffers, object pools, and bounded work per frame.
- Keep note rendering allocation-light; avoid analysis in `OnNoteMove()`.
- Define cache lifetime/invalidation before adding a cache.
- Include chart identity, mode, rate, judge scale, modifiers, key count, algorithm version, and every result-affecting option in cache keys.
- Invalidate caches when their inputs or lifetime change. Retry resets per-attempt derived state; immutable chart data may be reused only when its full identity/configuration key still matches.
- Keep an uncached fallback when cache validity cannot be proven.
- Measure initialization separately from frame-time work.

## Temporal state safety

- Name the clock that owns each timeline, rolling window, interpolation, animation, graph, and time-indexed cache. Do not silently mix gameplay time, audio time, wall time, or transformed chart time.
- Assume gameplay-facing clocks may move backward on pause/resume, seek, replay, retry, reload, or host correction unless target evidence proves otherwise.
- Define one reset/rebase function per temporal subsystem. It must cover every derived field owned by that clock: prior timestamps, cursors, ring heads, histories, samples, smoothing values, cached extrema/scales, animation bounds, and generated presentation geometry.
- Invoke the reset/rebase before consuming values from the regressed timestamp. If a lower layer resets its counters, reset dependent Lua/UI state in the same update path so stale high-scale history cannot be rendered against a reset low-scale denominator.
- Clamp or reject non-finite and out-of-range geometry after state transition, but treat that as a safety boundary rather than a substitute for resetting the corrupt source state.
