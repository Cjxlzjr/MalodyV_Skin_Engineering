# Runtime and Deployment

## Probe the host before selecting a deployment form

Treat the Lua host as a capability profile, not a stock Lua installation. A successful local Lua test or generic large-chunk compile test does not prove that the Malody script host accepts the same entry, globals, or load timing.

Use a **small UTF-8-without-BOM entry probe** in the exact skin directory and mode before deploying a protected or multi-file loader. Write a stage marker from `Init()` to known Composer text components. Interpret results in order:

1. No `Init()` marker: verify metadata-selected entry filename/path, skin reload/restart, mode, duplicate files, and BOM before loader logic.
2. `Init()` but no `OnHit()` marker: verify a real judged manual hit; AutoPlay, replay, miss-only input, or the active mode may not invoke `OnHit()`.
3. `OnHit()` marker: only then probe optional module loading, script-source paths, and callback-dependent capabilities.

Probe in the real host and record exact version, platform, mode, and callback:

- **Encoding:** Some hosts silently reject a UTF-8 BOM even when a local parser accepts it. Require no BOM for production entries unless the exact host demonstrates otherwise.
- **Companion modules:** Never assume global `require`, `package`, `package.path`, `package.searchpath`, `debug.getinfo`, or a source path exposing the skin directory. Test the actual companion filename from the real entry. If `require` is unavailable, use a verified single-file deployment rather than repairing `package.path`.
- **Top-level timing:** `Module`, `Game`, Composer objects, or error UI may be unavailable during initial chunk evaluation and available in `Init()`. Top-level diagnostics cannot reliably report top-level failures. When allowed, defer optional/expensive loader work to `Init()` and surface the captured stage there.
- **Environment lookup:** Host globals may come through an environment metatable: `Game`/`Module` can resolve while `rawget(_ENV, 'Game')` is `nil`. For host APIs, use protected ordinary lookup such as `pcall(function() return _ENV.Game end)`, then validate the table/userdata and methods. Reserve `rawget` for globals deliberately installed as direct environment fields.
- **Global writes:** Independently test setting/clearing globals and whether a loader may replace `require` or callbacks. Successful `rawget` does not imply writes are permitted.

For protected loaders, test the exact static source, not merely a similarly sized generated chunk. If failure can occur before `Init()`, provide a small deferred diagnostic wrapper that captures the stage and displays it after Composer is available. Never release that wrapper or embedded trust material.

### Recorded Free Mode `load` observation - July 17, 2026

Treat this as constrained, not Malody-wide:

- Deployment: loose `D:\muziskin\doublejudge.v`, active entry `doublejudge.lua`.
- UI: from `Init()`, write status through `Module:Find` to existing text components `acc_target1_text` and `acc_target2_text`.
- Result: `VMP6 load: OK`, `env=42, large=4096`.
- Verified: resolve host `load`; compile a text-only chunk with a supplied environment; execute it with environment state; compile/execute generated source with 4,096 increments (about 57 KiB).
- Implication: permit a VMP6 dynamic loader calling `load(source, chunkname, "t", environment)` for this exact Free Mode target.
- Repeat after changing Malody version, OS, skin, entry, Composer preview, Basic/Play/Free mode, or generated-stage size.
- Fail closed if `load` is missing, rejects text mode/environment argument, or cannot compile the selected size; never silently downgrade a protected release.

## Lifecycle and state ownership

Map callbacks before changing behavior:

- `Init()` - bind Composer modules; read configuration; initialize caches, pools, queues, timers, and empty/loading UI.
- `Update()` - bounded frame work, animation, dirty UI, timers, and scheduled calculations.
- `OnHit()` - consume one engine judgment event, validate fields, update feedback/statistics once.
- `OnInput()` / `OnKey()` - confirmed press/release semantics and input-to-lane mapping.
- `OnRetry()` - clear every mutable chart-dependent table, cursor, queue, pool, timer, cache, and derived display value.
- `OnDrawNote()` - select note modules and create visuals without full-chart scans.
- `OnNoteMove()` - update an existing note using documented progress/list representation.
- Free Mode processing callbacks - use only documented parameters and permitted operations.

For each mutable state, record owner, unit, lifetime, reset point, and source of truth. Keep callbacks thin; move pure calculations/state transitions into reusable functions or modules.

## Encoding and source hygiene

- Write production Lua as UTF-8 without BOM unless the target requires another encoding.
- Never convert through an unknown system code page.
- Preserve Chinese Composer names and user-facing runtime strings exactly.
- Keep comments short/readable; default to English for universal maintenance unless the project requires another language.
- Scan for mojibake, private-use Unicode, replacement characters, malformed long comments, and accidental literal changes.
- Never "repair" strings, Composer keys, or asset names by visual similarity; compare exact Unicode code points with known-good files.
- Avoid duplicate configuration keys. If legacy duplicates remain, document which assignment wins.

## Blank or load-failure diagnosis

Use this order:

1. Check encoding, BOM, malformed comments, and accidental string corruption.
2. Confirm active entry and duplicate-file selection.
3. Reproduce `require` loading from the deployed skin directory.
4. Confirm every dependency, data file, model, asset, and Composer element.
5. Confirm `Init()` completion and the last initialized subsystem.
6. Check `Module:Find`, Composer types, clone names, property writes, and parent relationships.
7. Check callback names, parameters, mode restrictions, and version gates.
8. Verify coordinate root, dimensions, anchors, alpha range, draw order, units, indices, masks, and enums.
9. Disable optional UI, effects, and analysis one subsystem at a time while preserving core gameplay.
10. Confirm `OnRetry()` resets all mutable and derived state.

When runtime data is missing, request exact MalodyV version/platform, mode, entry, callback, error/log, Composer setup, and minimal reproduction. Never guess undocumented API behavior.
