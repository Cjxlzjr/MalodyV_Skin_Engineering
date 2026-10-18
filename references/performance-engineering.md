# Performance Engineering

Use for a measured hot path or a requested performance investigation. Local profiling and deterministic work can proceed without the game host; claims about FPS or host improvement require comparable host measurements. Examples below are scoped fixtures, not mandatory rates for every skin.

Use this reference for high-density frame drops, excessive `Update()` work,
Composer write storms, Lua/native call fan-out, report overhead, or a measured
performance migration. Keep gameplay semantics and performance evidence as two
separate contracts.

## Performance contract

Before comparing performance, record the applicable baseline fields below. Missing host measurements block host improvement claims, not independent local instrumentation or a reversible source fix:

```text
target version/platform/mode/key count/resolution
active source and deployed copy
VSync/FPS cap/window mode
repeatable chart and exact dense interval
required event-immediate behavior
continuous displays and acceptable fixed refresh tiers
gameplay/report/reset contracts that must remain identical
baseline metrics and exact-host FPS procedure
```

Classify numbers correctly. Callback counts and elapsed time from a deterministic
probe are `probe` evidence. FPS from the pinned game host is `exact-host`
evidence. A reduction in simulated work does not prove a target FPS.

## Hot-path inventory

Trace work by frequency, not only by file:

| Frequency | Typical work | Default treatment |
|---|---|---|
| Event-immediate | key feedback, judgment, score transition, report hit record | Keep immediate; normalize the event once |
| Visual continuous | graph geometry, fading judgment, interpolation | Fixed cadence such as 120 Hz; lower only by an explicit tier |
| Ordinary UI | score text, ACC text, MM, auxiliary labels | Dirty bit or a fixed cadence such as 60 Hz |
| Low-frequency observation | wall clock, report clock, non-urgent polling | Fixed cadence such as 20 Hz |
| Finalization | complete result snapshot, JSON/HTML export | Lazy; construct only after the finish condition |

The frequencies above are examples, not MalodyV constants. Choose them from the
visible requirement and validate the chosen tier on the exact target. Data-state
advancement and Composer presentation may use different cadences, but define
which values may lag and the maximum visible latency before splitting them.

Within one callback, reuse one coherent event/time snapshot unless distinct reads are required by a verified contract:

```lua
function Update()
    local time_ms = Game:Time()
    update_runtime(time_ms)
end

function OnInput()
    local event = Game:InputEvent()
    if not event then return end
    local input_type = tonumber(event:Type()) or 0
    local lane = tonumber(event:HitX())
    local time_ms = Game:Time()
    handle_input(input_type, lane, time_ms)
end
```

Pass normalized scalars to every subsystem. Do not let descendants reread
`Game:Time()`, `Game:InputEvent()`, or `Game:HitEvent()` for the same callback.

## Drift-free fixed cadence

Do not schedule repeated work with `next = now + interval`. On integer-millisecond
clocks, that drifts downward: an intended 120 Hz interval of `8.333... ms` can
become one update every 9 ms. Advance from the prior deadline and skip missed
slots without running catch-up bursts:

```lua
local function cadence_due(now_ms, state, key, interval_ms)
    local deadline = state[key]
    if deadline == nil then
        state[key] = now_ms + interval_ms
        return true
    end
    if now_ms < deadline then return false end
    local steps = math.floor((now_ms - deadline) / interval_ms) + 1
    state[key] = deadline + steps * interval_ms
    return true
end
```

Reset every deadline on Retry, reload, replay restart, mode/chart transition,
or clock regression. A future deadline retained after rollback can suppress UI
for an entire replay episode.

Keep catch-up bounded. Replace a `while now > next_tick` loop whose cost grows
with a pause or seek by calculating missed ticks. If the missed count exceeds a
ring's capacity, compute the same final ring state while touching at most one
ring capacity of slots.

Keep quality tiers fixed and reproducible. If exact-host acceptance misses the
target, switch the declared continuous-visual tier (for example 120 Hz to
60 Hz), rerun the same validation, and record the choice. Do not silently
disable features or introduce dynamic degradation unless the user requested it.

## Composer write suppression

Treat a Composer property assignment as a host boundary call, even when the Lua
calculation is cheap.

- Write immutable `Height`, base alpha, dimensions, and clone layout only during
  initialization or a real layout invalidation.
- Cache the last submitted `X`, `Y`, `Width`, `Height`, `Rotate`, `Alpha`,
  `Value`, and `Text` per node.
- Submit a property only when its visible value changed; use a target-appropriate
  epsilon for floating geometry.
- Recompute multi-segment geometry only when its cadence is due. Do not redraw
  the same graph twice through separate scale and scroll branches.
- Invalidate the presentation cache together with its producer on rollback and
  reset.

Dirty groups from native code do not replace per-property caching when a dirty
group still contains hundreds of unchanged nodes.

## Lazy report and provider boundaries

Do not construct an expensive value before the consumer knows it is needed.
Use providers for wall-clock reads and final snapshots:

```lua
report.update(
    time_ms,
    function() return build_result_snapshot() end,
    function() return Game:SystemTime() end
)
```

The report invokes the wall-clock provider only on its sampling deadline and
the result provider only during finalization checks. Preserve hit recording and
schema fields independently of the sampling cadence.

Downsampling clock observation changes interval boundaries. When a sampled
wall-clock gap spans a `Game:Time()` rollback, do not count the rollback twice
as untracked pause time. Preserve raw deltas, merge correlated signals, and run
the existing threshold/boundary fixtures after changing cadence.

A prolonged rollback may contain hundreds of negative `dt` frames. Rebase the
sampling deadline once when entering the rollback episode; do not force a wall
clock read on every negative frame. Continue lightweight cumulative rollback
tracking, then end the episode on recovery, Retry, or reload.

## Lua/native event snapshot

After native code processes an event, prefer one fixed-order scalar return over
a fan-out of getters for judgment, head/tail state, combo, max combo, score,
accuracy, and counters.

- Reuse the existing export only when target evidence proves legacy callers
  ignore its return values; otherwise add a versioned snapshot surface.
- Document return order and scalar types as an ABI contract.
- Do not allocate a table per event merely to package the snapshot.
- Compare every returned field with the legacy getter state after every fixture
  event, including Tap, Hold head/body/tail, duplicate callbacks, Miss, and
  Retry.
- If the return layout must later change, add a versioned surface instead of
  inserting fields into an established order.

## Deterministic dense probe

Use the same normalized event stream before and after the edit. A useful probe
drives the expected host update rate and representative input/judgment rate,
then records:

```text
Update/Input/Hit counts
Composer writes by property
Game/InputEvent/HitEvent/SystemTime calls
native calls by function
Lua GC delta
elapsed time
frame-time median/p95/p99 or equivalent host percentile
per-event gameplay snapshot or first mismatch
reset/retry/rollback state
```

Compare the implementation against the preserved pre-change source, not only a
theoretical write-count estimate. Require zero gameplay-state differences for a
performance-only change. Set the required write/call reduction from the task;
do not turn one skin's percentage into a universal threshold.

When instrumenting Composer mocks, remember that Lua `__newindex` is not called
for keys already stored directly in a table. Use a proxy with a backing-value
table when every assignment must be counted.

## 🔴 CHECKPOINT · Performance

Do not claim a performance optimization is complete until all applicable rows
are resolved:

| Trigger | Required action | If unavailable |
|---|---|---|
| No repeatable dense fixture | Pin one chart/interval and settings before measuring | Stop the performance claim |
| No before/after counters | Add bounded instrumentation and preserve the baseline source | Report only a hypothesis |
| Gameplay snapshot differs | Stop at the first event and restore or isolate the change | Do not trade correctness for speed |
| Clock/report fixture differs | Inspect sampling boundaries and correlated rollback/gap accounting | Do not weaken thresholds silently |
| Native boundary changes | Run ABI/self-test, export inspection, per-event parity, and reset coverage | Keep the old path |
| Exact host unavailable | Complete static/focused/replay evidence and write the exact manual FPS procedure | Mark FPS acceptance `blocked` |
| Target FPS misses at the high tier | Select the predeclared lower fixed tier and rerun all affected checks | Do not auto-disable features |

## Performance anti-patterns

- Do not move arbitrary code to native before measuring the dominant cost.
- Do not rate-limit event feedback to solve continuous-UI overhead.
- Do not call host getters independently from every descendant module.
- Do not write unchanged Composer properties every frame.
- Do not allocate result tables or serialize reports every frame.
- Do not reset only the clock while retaining future deadlines, geometry, or
  cached scales.
- Do not use a faster mock as proof of exact-host FPS.
- Do not publish a best-run FPS; use the same interval and a declared statistic
  such as the median of three runs after a consistent warm-up.
