# Gameplay Clock and Pause Anomalies

Use for pause/clock detection or reports that interpret offsets around a suspected pause. Preserve raw timestamps and hit records; reset/rebase derived windows according to [temporal state safety](implementation-safety.md#temporal-state-safety). A backward clock alone is not a new attempt.

Use this reference when a MalodyV skin report shows a sudden timing-offset change, especially after the player pauses and resumes during a chart.

## Diagnostic Boundary

- **Do not start by changing judgment math.** First establish whether event timestamps changed context.
- **Do not equate replay behavior with live-play behavior.** Replay adapters may synthesize, quantize, reorder, or omit timestamps.
- **Do not equate a long chart gap with a pause.** Sparse patterns, missing callbacks, dropped events, seek/correction behavior, and speed changes are alternatives.
- **Do not claim “no pause” when details are unavailable.** Use `unavailable` and name the missing evidence.

## Evidence to Preserve

For every valid hit event, retain original values when available:

- chart/note time;
- game or `HitEvent` time;
- input or callback time;
- raw offset and normalized offset;
- judgment, event index, lane, and replay/live provenance.

Never overwrite original event values with corrected values. Keep derived analysis separate.

## Preferred Update-Level Detection

When the host exposes both `Game:Time()` and `Game:SystemTime()` in `Update`, sample them together. This is more reliable than inferring a pause from hit spacing because it observes the clocks even when there is no note event.

Use one consistent unit and record the observed unit in diagnostics:

```text
system_delta = system_now - system_previous
game_delta = game_now - game_previous
untracked = system_delta - game_delta

dual_clock_signal = system_delta > 0
                    and untracked >= minimum_untracked_ms

game_peak = maximum Game:Time reached in the current local episode
cumulative_regression = game_peak - game_now

regression_signal = cumulative_regression >= minimum_cumulative_regression_ms
```

The regression calculation must retain the local peak while the clock descends. Do not compare only adjacent updates: a pause-related correction can be distributed across hundreds of individually tiny negative steps. Advance the peak when game time reaches a new high; while below it, retain the peak and deepest trough. Close or rebase the episode only under an explicit recovery policy.

Create a candidate from either signal and merge signals that describe the same nearby transition into one pause interval. Record each constituent signal as well as the merged interval. On retry, reload, or a new play attempt, reset previous clocks, peak/trough state, active candidates, and correlation state atomically.

The locally validated threshold is `1000 ms` for both untracked time and cumulative regression. It is not a universal MalodyV constant. If `Game:SystemTime()` is unavailable, mark the dual-clock signal unavailable and retain cumulative game-time regression when possible. If neither clock stream is sufficient, report `unavailable`, not `not_detected`.

Do not export every update sample in a production report. Keep compact counters, resolution observations, candidate intervals, maxima, thresholds, and signal provenance; raw per-frame samples can make the HTML report unnecessarily large.

## Hit-Event Fallback and Corroboration

Use the following pattern only as a fallback when update-level clocks were not captured, or as corroborating evidence. It is not the primary pause oracle on the validated Windows Basic Mode target.

1. Filter to events with valid judgment and the clocks required by the probe.
2. For adjacent events, calculate:

   ```text
   game_gap = game_time[i] - game_time[i-1]
   note_gap = note_time[i] - note_time[i-1]
   extra_gap = game_gap - note_gap
   ```

3. Mark a candidate only when the gap is materially larger than normal chart spacing and record the threshold with the result.
4. Compare a small pre-gap and post-gap window of offsets. Record a median or robust center and:

   ```text
   offset_shift = post_center - pre_center
   ```

5. Raise confidence only when clock-gap evidence and a sustained post-gap offset shift support the same boundary. Otherwise use lower confidence and call it a candidate.

Numeric thresholds are project- and host-specific. Do not copy them as universal MalodyV constants; calibrate against normal sparse charts and record them in the report.

## Exact-Target Evidence: Steam Windows x64 Basic Mode

Evidence class: exact-host gameplay probe. Build number was not independently established, so do not generalize these values beyond the tested host without another probe.

| Field | Accepted observation |
|---|---|
| Entry/callback path | `mutsumi.lua` → `OsuJudge.update` → `play_report.update`, sampled in `Update` |
| Fixture | Manual pause near the end of `Soleily - Violet sphere [4K - Stamina Lv.30]` |
| Report | Schema `3.3.0`, generated 2026-09-13, 505 hit events |
| System clock | Available, monotonic in the captured transition, observed `1 ms` resolution |
| Dual-clock signal | `system_delta=4443 ms`, `game_delta=1 ms`, `untracked=4442 ms` |
| Cumulative regression | `2040 ms`, local `Game:Time` peak `11555` to trough `9515` |
| Detector result | One merged, high-confidence interval with two signals: `dual_clock_gap` and `cumulative_game_time_regression` |
| Integrity | Exported once; Malody and osu event totals matched; no report warnings |

An earlier diagnostic run measured a `3040 ms` system/game gap followed by `2047 ms` of cumulative rollback across `294` negative updates. Its individual negative steps were about `1–47 ms`. This is the decisive reason to use peak-to-trough accumulation instead of a per-update `1000 ms` rollback test.

### Data behavior across the accepted pause

The clocks and gameplay records did not all move backward in the same way:

| Data stream | Observed behavior |
|---|---|
| Update-level `Game:Time()` | Fell cumulatively `2040 ms` after the dual-clock gap |
| `Game:SystemTime()` | Continued forward; no captured regression |
| Hit-event game time | One small regression only: `11544 → 11519` (`-25 ms`) |
| Hit-event note time | No regression; boundary values advanced `11430 → 11525` |
| Boundary judgment | Event 151 was `Miss`, then event 152 was `Perfect`; do not infer causation from one run |
| Combo/score | Miss reset combo; the next hit resumed combo and score progression normally |
| Event totals | 505 events, one miss, with integrity checks passing |

For descriptive analysis only, the 20 valid hits immediately before the boundary had mean offset `-7.05 ms`, median `-8 ms`, and mean absolute error `16.15 ms`; the next 20 valid hits had mean `+8.85 ms`, median `+10.5 ms`, and mean absolute error `12.75 ms`. Report this as a local offset-window change, not proof that pause caused the shift. Player timing, the small sample window, chart structure, and other host behavior remain alternatives.

This evidence establishes that the detector ran normally on this target: two independent update-level signals identified one interval, while the event stream stayed internally consistent. It also establishes that hit-event or note-time rollback alone would be an unreliable pause detector here.

## Report Requirements

The UI should show, in text as well as color:

- no pause detected, possible pause detected, or unavailable;
- number of candidates;
- before/after event identifiers;
- start/end time and extra gap;
- offset shift when computable;
- confidence and data source;
- a limitation note explaining that detection is heuristic and does not change judgment accuracy.

Keep the component responsive, keyboard-readable, and safe when `events` is empty or partially populated. Optional timing fields must not make report generation fail.

## Validation Matrix

At minimum, test:

| Case | Expected |
|---|---|
| normal dense play | no false pause candidate |
| normal sparse chart | no automatic pause claim |
| manual pause/resume with clock gap | candidate interval, thresholds, clock deltas, and signal provenance are visible |
| gradual cumulative rollback | peak-to-trough total triggers even when every individual negative step is below threshold |
| correlated gap and rollback | one merged pause interval containing both signals |
| pause followed by offset shift | shift is reported separately from original offsets |
| missing timing fields | unavailable, not “clear” |
| replay-generated events | provenance is retained and live-play equivalence is not claimed |
| hit details disabled | report remains valid; hit-based corroboration is unavailable, but independently sampled Update-clock detection may remain available |
| retry/reload/new attempt | no stale candidate state leaks across attempts |
| rollback within the same attempt | retain peak/trough and merged-candidate evidence until the defined recovery boundary; do not erase the signal as if Retry occurred |

Static validation proves serialization and fallback behavior only. A real pause/resume acceptance claim requires an exact-host or controlled gameplay probe with the active deployed copy identified.
