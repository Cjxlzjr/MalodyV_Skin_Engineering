# Validation

Select checks by changed behavior. These are routes, not a full-suite requirement for every edit. Run repository-required checks applicable to the change; broaden only for a failure, new change, or unresolved risk.

## Scope selection

| Change | Low-cost local evidence | Target evidence when relevant |
|---|---|---|
| Skill/reference prose | Frontmatter/JSON, paths/anchors, changed claims against sources, trigger positive/negative cases | No gameplay run for prose alone |
| Lua source | Target-compatible parser, UTF-8 without BOM, changed callback/signature and return contracts | Runtime assumptions being changed |
| Entry/loading/dependency | Isolated entry harness; changed reachable require edges and required resources | Actual entry selection and host load logs |
| UI/configuration | Value/format/absence cases, affected property ownership and state reset | Relevant node/setting in Composer and game |
| Clock/cache/animation | Non-zero rollback fixture below, reset/rebase and bounds | Pause/resume, seek, Retry or replay path affecting this clock |
| Gameplay/algorithm | Normalized fixtures and per-event oracle, applicable notes/mods/units | Manual/Auto/replay/finish paths affected by the change |
| Native/performance | Relevant ABI/self-test/replay or fixed-workload measurements | Exact-host load/smoke or comparable host measurements before activation/claims |
| Deployment | Backup, exact files, source/destination hashes, rollback | Active deployed entry and visible behavior |

For changed Lua, inspect unsupported syntax/libraries, missing module returns and accidental duplicate definitions/globals on the affected path. Do not require loading the full skin once per changed file. A harness with host mocks proves only simulated behavior.

## Focused fixtures

Choose cases that can expose a failure in the changed contract:

- Chart parsing: empty/one-note chart, lane/index boundaries, chord/hold, timing points and affected rate/mod transforms.
- Judgements: perfect/early/late/Miss, duplicate callback, out-of-window input; long-note phases use the [event fixture](player-gameplay-rules.md#long-note-event-and-accuracy-rules).
- UI: zero/first/large value, empty/long/localized text, missing optional node or invalid required setting, affected animation lifecycle.
- State ownership: Retry/reload/mode transition and time rollback where those paths own changed state.
- Metrics: sample thresholds, fixed offsets, all-Miss input, outliers, truncated windows and cross-mode comparison where promised. Distinguish chart difficulty from player performance; compare intermediate fields and avoid transforming both chart time and rate twice.

Assert observable values, counters, event order, formatting, node properties and resets. Absence of errors is insufficient.

## Time-regression checks

For a changed clock-driven graph, animation, rolling window, timeline or cache:

1. Seed non-zero state above any minimum display scale.
2. Advance multiple updates until cursors, histories, smoothing, extrema and geometry are observable.
3. Move the same clock backward without Retry, unless Retry is the behavior under test.
4. Continue forward and exercise Retry separately.

Require no stale future-owned derived sample/cursor/scale/animation bound; reset or rebase all dependent Lua/native/UI state in the same lifecycle path. Values must remain finite, recover on forward updates, and keep geometry inside verified owner/container bounds. Preserve raw observations and deliberate pause-detector peak/trough evidence according to their separate lifetime; do not clear a detection episode because its signal is rollback.

A zero-only history cannot expose stale-history divided by reset-scale bugs. Capture the pre-fix out-of-bounds property or stale state and rerun the same fixture after repair. Clamping geometry is a final guard, not a substitute for correcting state.

## Evidence and unavailable checks

For each materially changed behavior, record fixture/action, expected, observed, evidence level and result. A short paragraph suffices for a small edit; use a table when cases differ.

- `pass`: observed expectation at the stated level (static, deterministic, simulator or host).
- `fail`: observed mismatch; retain the first divergent event or property.
- `unverified/blocked`: required host, fixture, log or oracle unavailable; name what is missing.
- `not applicable`: outside the change.

Complete independent authorized work despite unavailable host access. Supply a host procedure with version/platform/mode, active copy, fixture, action, expected visible result and relevant log; do not manufacture acceptance. Deployment requires its own authorization and checks.

## Skill behavior comparison

For instruction changes, compare an in-scope task and a small out-of-scope task with the same model, reasoning effort, tools, user input and raw artifacts; vary only the Skill version. Record what was executed. Static trigger review, an agent's dry-run response, and end-to-end execution are different evidence levels. Do not infer speed, quality or token improvement from file size alone.
