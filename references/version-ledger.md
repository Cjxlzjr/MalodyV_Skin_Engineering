# Version and Changelog Ledger

Use this ledger for post-6.2 compatibility. Source priority remains official API documentation, then official MalodyV changelogs, then exact-target observations. Baseline checked July 18, 2026: Windows/Steam `6.7.12` (announcement date appears as June 28 or June 29 by locale/time zone).

## Skin/Composer changes

| Version | Capability/change | Scope and status | Required handling |
|---|---|---|---|
| 6.2.22 | Sound Module | Composer skin module; changelog-confirmed, detailed properties absent from bundled API docs. | Inspect the target module panel; do not substitute `Audio:` behavior without evidence. |
| 6.3.32 | Bar Module for Key/Slide | Composer module; changelog-confirmed. | Confirm value range, direction, layer, and script-visible properties in the target. |
| 6.4.2 | TouchArea Module; edit basic parameters of default modules; edit existing Scene/Trigger | Composer/editor capabilities, not automatically Lua APIs. | Do not call them as globals; inspect generated modules/scenes/triggers. |
| 6.4.2 | `Game:WriteFile`, `Game:WriteFileChart` | Skin Lua names confirmed by changelog; bundled docs omit signatures. | Gate by version/capability. Target case confirms `Game:WriteFile(path, content)` only; do not invent `WriteFileChart` parameters. |
| 6.5.2 | `Game:PlayMeta` / `Game:PlayMeta("speed")` | Skin Lua name confirmed; generic key/return schema remains target-specific. | Runtime observations indicate that supported hosts may return `speed` as a positive effective-speed multiplier. Verify the key, type, value range, callback availability, and unavailable-key behavior on the exact target; do not combine a valid value with Mod-bit parsing or Composer Turbo correction. |
| 6.5.32 | `OnFinish` | Skin Lua callback name confirmed by changelog. | Do not assume parameters, exact delay, Composer behavior, retry behavior, or single invocation without a probe. |
| 6.6.2 | Video custom removal color | Composer video capability. | Confirm color format, tolerance, platform codec, and whether it is script-visible. |
| 6.6.2 | Fixed `Module.Alpha` getter always returning `0` | Runtime bug fix. | Treat reads on older versions as unreliable; cached original alpha may be safer after a target test. |
| 6.6.22 | `Game:FileHash` | Skin Lua name confirmed; signature/algorithm absent from bundled docs. | 6.7.12 probe found it callable, but two unverified names returned `""`; path scope, missing-file result, hash format, and Composer behavior remain unknown. |
| 6.6.43 | `Network:DoRequest` / `OnResponse` | Free Mode skin API only; names confirmed by changelog. | 6.7.12 probe: `Network:DoRequest` is a function while `Network.OnResponse` is `nil`, supporting a global callback interpretation; signature/trigger remain unconfirmed. Require explicit network intent and bounded error handling. |
| 6.7.12 | No new skin Lua name in changelog; timing, osu!mania key-sound, and SV fixes | Compatibility baseline. | Re-test scripts that compensate for speed-mod timing, key sounds, or osu!mania SV; remove compensation only with before/after evidence. |

## Documentation discipline

- Never derive a callable signature from a changelog name alone.
- Label each fact as **official documented**, **official changelog name**, **target-observed call**, or **runtime-probed behavior**.
- Use `Game:IsVersionGE` where documented, plus protected capability lookup when newer APIs may be absent. A version check does not prove callback availability or argument shape.
- Keep skin APIs (`Game`, `Module`, `Audio`), Free Mode APIs (`Chart`, `Note`, `Play`, `Network`), Composer module types, and Editor plugin APIs separate.
- Do not treat `Editor:DoRequest`, editor `OnResponse`, `WriteFilePlugin`, or file-selection plugin APIs as skin Lua capabilities.
- For file/network APIs, verify path sandbox, encoding/binary behavior, overwrite rules, size limits, failure return, privacy, and whether calls are allowed in Composer, replay, AutoPlay, multiplayer, or result flow.

## Minimum runtime probes

- `WriteFile*`: write a tiny unique file, read/verify it if supported, and confirm target directory and overwrite behavior.
- `PlayMeta`: record known keys, Lua types, values under no Mod and speed Mods, and unavailable-key behavior.
- `OnFinish`: mark invocation time/count across normal finish, retry, quit, replay, AutoPlay, and empty charts.
- `FileHash`: hash a known small asset and a missing file; record format and determinism.
- `Network`: test success, timeout, malformed response, non-2xx response, duplicate callback, response-size cap, and retry/reset cleanup without exposing secrets.
