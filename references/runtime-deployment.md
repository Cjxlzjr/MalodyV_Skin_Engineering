# Runtime and Deployment

Use loader sections for loading failures, file-output sections for reports, and deployment checks only for a requested installation. A source fix does not authorize copying to a live skin. If source/destination and backup scope are already authorized and unambiguous, do not ask for confirmation again.

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

## File output and JSON reports

### Target-observed `Game:WriteFile` contract — August 13, 2026

On the tested Windows MalodyV host, Lua calls `Game:WriteFile(relative_path,
content)` directly; no skin DLL is required for JSON encoding or writing.
Successful calls returned `nil`, so success is `pcall(...) == true` **and** the
returned value is not explicitly `false`.

- Paths are relative to the active skin root.
- The host does not create parent directories. Create `output_report/` in the
  skin before runtime; otherwise a write to `output_report/name.json` throws a
  `DirectoryNotFoundException`.
- Use a primary `output_report/<sanitized-name>.json` path and a skin-root
  fallback `<sanitized-name>.json`; record `fallback_used` and a compact error
  reason in the report.
- Sanitize Windows-invalid filename characters `\\ / : * ? " < > |`; trim trailing
  dots/spaces and handle an empty result.
- Treat a DLL as a computation boundary only. Native code may supply metrics,
  but Lua owns `Game`, `Module`, Composer, JSON serialization, lifecycle, and
  file output unless target evidence proves another design.

### Completion export: August 13 probe strategy

In this probe, the host could switch away from the play scene immediately after audio end. The probe therefore did not wait for `AudioLength()+200 ms`, multiple delayed 100 ms samples, or a
later result-screen callback. Its `Update()` exported once on the first observed
frame where `Game:Time() >= Game:AudioLength()`; guard with `finalized` before
building or writing the report. Exact-host evidence recorded a successful write
at `60601 ms` for a `60600 ms` chart immediately before `Switch to 6`.

Keep periodic snapshots separate from finalization. Event-forced samples are
useful for changes, but finalization must not depend on their cadence. For
this probe, Retry retained prior history in a new run segment. Production reports
may instead clear the previous attempt; preserve the target schema and retention
policy. Log clock rollback separately from Retry.

Do not replace a target's settled-score finalizer with this first-frame strategy
without comparing final counters and scene timing. Mutsumi's current report path
has used delayed/stable-snapshot finalization and cleared Retry history; these
are distinct contracts. Verify the active implementation and exact-host finish
behavior before changing either policy.

### JSON validity requirements

- Encode strings byte-for-byte with JSON escapes for quote, backslash, tab,
  LF, CR, backspace, form feed, and every remaining `0x00..0x1F` control byte.
- Host exception text can contain CRLF stack traces. Never insert it unescaped
  into a JSON string; prefer a concise error summary when full stack text is
  not needed.
- Validate the generated file with a strict JSON parser after exact-host export,
  not merely with a Lua encoder or a log message.
- Log explicit milestones: `Init reached`, primary write result, fallback write
  result, and finalize time versus audio length. This distinguishes inactive
  entry, missing directory, failed write, and missed finalization.

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

Start with active-entry selection, then investigate the first failing stage. Use the relevant checks below, not an unconditional ten-step mutation sequence:

1. Check encoding, BOM, malformed comments, and accidental string corruption.
2. Confirm active entry and duplicate-file selection.
3. Reproduce `require` loading from the deployed skin directory.
4. Confirm dependencies, data files, models, assets, and Composer elements on the failing reachable path.
5. Confirm `Init()` completion and the last initialized subsystem.
6. Check `Module:Find`, Composer types, clone names, property writes, and parent relationships.
7. Check callback names, parameters, mode restrictions, and version gates.
8. Verify coordinate root, dimensions, anchors, alpha range, draw order, units, indices, masks, and enums.
9. For an authorized fix or isolated probe, disable a suspected optional subsystem with a real fallback. Pure review/diagnosis requests remain read-only.
10. Confirm `OnRetry()` resets all mutable and derived state.

Collect available local evidence first. Request only missing result-changing details among exact MalodyV version/platform, mode, entry, callback, error/log, Composer setup, and minimal reproduction. Never guess undocumented API behavior.
