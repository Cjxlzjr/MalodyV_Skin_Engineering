# Validation

## Static checks

For every changed Lua file:

1. Parse with the target Lua runtime/parser.
2. Verify UTF-8 without BOM; scan for mojibake.
3. Load the top-level entry from the actual skin directory.
4. Walk reachable `require` edges; report missing modules.
5. Check duplicate functions, undefined globals, unsupported syntax/standard-library calls, and missing module returns.
6. Compare engine-facing names/callback signatures with the applicable official reference.

## Focused behavior checks

Start small, then broaden:

- Empty and one-note charts.
- Tap, hold, release, break, chord, jack, dense stream, and mixed patterns.
- Multiple key counts and lane-index conventions.
- Perfect, early, late, Miss, duplicate callback, and out-of-window input.
- Manual, AutoPlay, replay, Retry, reload, and mode switching.
- Missing Composer modules/assets and invalid configuration.
- Default/non-default rate, modifier, judge scale, and timing points.
- Long titles, empty labels, zero/few samples, and maximum pool pressure.

Verify observable behavior, not just absence of errors: returns, labels, positions, alpha, animation, score counters, event order, note completion, reset state, algorithm route, and cache invalidation.

## Algorithm and measurement checks

For difficulty/player statistics:

- Separate chart difficulty from player performance.
- Inspect normalized inputs before comparing outputs.
- Test fixed-rate changes, fixed-offset inputs, all-Miss input, and identical charts under different mods.
- Confirm transformed versus raw chart time; never transform both chart and rate unless the reference requires it.
- Check sample thresholds, outliers, Miss treatment, chart-window truncation, and cross-mode comparability.
- Compare intermediate fields, not only final labels/scores.
