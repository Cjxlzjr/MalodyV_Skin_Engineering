# Reference Implementation Module Provenance Map

This map records which verified source areas in the local reference implementation informed the reusable module catalog. Use it to inspect implementation ordering and state ownership. It is not a copy/paste map: all Composer IDs, Chinese settings keys, node geometry, assets, and Native assumptions belong to the reference implementation unless independently verified in the target skin.

## Encoding and source handling

- The main entry, runtime, configuration, locale, judgement, and metric Lua files are UTF-8 source files.
- Preserve UTF-8 when reading or editing them. Console output that uses a non-UTF-8 code page can render Chinese Composer IDs or setting keys as mojibake; that is not evidence that the source is invalid.
- `info.asm` is serialized/binary data. Do not treat it as a normal UTF-8 script or rename its internal strings from a text dump.
- The optional Native module is a binary dependency. This map documents only the Lua-visible contract and probe evidence; it does not infer internal algorithms.

## Runtime path

```text
public callbacks in the main entry Lua file
  Init     -> runtime.init      -> init_runtime
  Update   -> runtime.update    -> update_runtime
  OnHit    -> runtime.on_hit    -> on_hit_runtime
  OnInput  -> runtime.on_input  -> on_input_runtime
  OnRetry  -> runtime.on_retry  -> on_retry_runtime
```

The runtime adapter provides initialization gating and handler dispatch. The main entry Lua file constructs the ordered feature graph. The observed order is important because later feature setup reads shared data and Native state created earlier.

## Infrastructure layer

| Reusable module | Reference source | Runtime path | Project-specific dependencies | Generalization boundary |
|---|---|---|---|---|
| Lifecycle dispatcher | `runtime.lua`: `configure`, `init`, `update`, `on_hit`, `on_input`, `on_retry` | All callbacks | Handler names only | Reuse directly with target handlers; do not assume the target needs a dispatcher. |
| Shared play state | Main entry: `InitSharedData`, `UpdateSharedData`, `GetL` | Init, Update | Chart metadata fields, existing score/ACC text nodes, mod-info text | Preserve the target's authoritative score/accuracy source and chart metadata names. |
| Configuration adapter | `config.lua`: `read`, `get_bool`, `get_number`, `get_string`, `read_locale` | Init and feature setup | Chinese Composer setting keys | Reuse safe-reading pattern only; target keys and defaults must be discovered. |
| Locale adapter | `locale.lua`: `init`, `get`, `format`, `kps_window` | Init and formatting | Reference locale table | Reuse table/fallback pattern; target language selection and text keys are independent. |
| Static node hiding | Main entry: `STATIC_HIDE`, `STATIC_HIDE_Execute` | Init, Retry | Named reference nodes | Reuse optional cached visibility pattern after confirming which target nodes may be hidden. |
| Display cache | Main entry: `last_*` fields, `UpdateSharedData`, `MM_updateDisplay`; judgement module: last display keys | Update, Retry | Existing reference text values | Reuse cache-and-write-on-change behavior; formatting and units remain target-specific. |

## Interaction and animation layer

| Reusable module | Reference source | Runtime path | Project-specific dependencies | Generalization boundary |
|---|---|---|---|---|
| Multi-instance animation state | Main entry: `KEYOVERLAY.ani_*` state arrays and update logic | Init, Update, Input, Retry | Reference node geometry and factors | Reuse per-instance base/transient-state design; recalculate target anchors and coordinate root. |
| Key overlay discovery | `KEYOVERLAY_getConfigStatus`, `KEYOVERLAY_findUIElements` | Init | ChartInfo key count, reference setting keys and IDs | Verify target key count, lane order, optional settings, and templates. |
| Clone management | `KEYOVERLAY_cloneOverlay`, `KEYOVERLAY_cloneKeyImage_KeyImage_Letters_Counts` | Init | `Module:Clone` templates and generated reference IDs | Clone once at initialization; target must provide cloneable templates and collision-free names. |
| Key overlay layout | `KEYOVERLAY_initOverlay`, `KEYOVERLAY_initKeyImage_Letters_Counts`, `KEYOVERLAY_hideKeyImage_Letters_Counts`, `KEYOVERLAY_Init`, `KEYOVERLAY_Reset` | Init, Retry | Overlay bar dimensions, image sizes, text nodes | Preserve base coordinates independently of animated values; no fixed reference dimensions. |
| Input-driven effects | `KEYOVERLAY_HandleInput` | OnInput | Event-to-lane semantics, speed-corrected duration | Verify input type/lane mapping and time unit in the exact mode. |
| KPS graph setup | `KPS_CHART_getConfigStatus`, `KPS_CHART_findElements`, `KPS_CHART_cloneElements`, `KPS_CHART_initState` | Init | Reference graph IDs, KPS settings, templates | Reuse bounded history and clone-once strategy; graph geometry and display mode must be target-specific. |
| KPS graph rendering | `KPS_CHART_resetBarsPosition`, `KPS_CHART_moduleVisibility`, `KPS_CHART_rescaleHistogram`, `KPS_CHART_drawLines`, `KPS_CHART_Update`, `KPS_CHART_HandleInput`, `KPS_CHART_Init`, `KPS_CHART_Reset` | Init, Update, OnInput, Retry | Histogram/line nodes, expected input set | Preserve bounded work and reset behavior; do not assume bar/line property semantics. |
| Clock/progress | `CLOCK_getConfigStatus`, `CLOCK_findElements`, `CLOCK_initBar`, `CLOCK_moduleVisibility`, `CLOCK_updateBar`, `CLOCK_Init`, `CLOCK_Reset`, `CLOCK_Update` | Init, Update, Retry | Clock/bar node IDs and orientation | Verify duration source, direction, base width/height, and visibility convention. |

## Gameplay data and visual adapters

| Reusable module | Reference source | Runtime path | Project-specific dependencies | Generalization boundary |
|---|---|---|---|---|
| MM/S presentation | `MM_findElements`, `MM_moduleVisibility`, `MM_syncNative`, `MM_updateDisplay`, `MM_Init`, `MM_Reset`, `MM_Update` | Init, Update, OnHit, Retry | Native MM API, mod text, reference nodes | Reuse source->format->cache->node separation. Do not reuse metric calculation without the target algorithm/source. |
| Dual accuracy | `DUAL_ACC_Init`, `DUAL_ACC_Reset`, `DUAL_ACC_OnHit` | Init, OnHit, Retry | Native dual accuracy API and custom judgement settings | Verify result enum and profiles. Native is optional only if a target-proven fallback exists. |
| Osu configuration | `OsuJudge.lua`: `read_play_speed`, `get_od`, `build_windows`, `config_snapshot` | Init, Native initialization | OD, speed, Score v2, HR, tail-judge settings | Preserve explicit settings/source mapping; do not infer judgement windows for another skin/mode. |
| Osu UI binding | Judgement module: `init`, `set_osu_nodes_visible`, `set_combo_text` | Init | Large reference node family | Reuse required/optional binding and cached writes, not node IDs or visual layout. |
| Judgement FX and offset bar | `update_judge_fx`, `update_offset_bar`, `native_hit` | OnHit, Update | Native judge mapping, reference animation nodes | Verify offset sign/unit, enum mapping, alpha/X semantics, and per-result node availability. |
| Score/ACC/combo smoothing | `native_sync`, `update`, `reset_state` | Update, OnHit, Retry | Native result source, display curves, formatting | Reuse explicit source/cache/reset pattern; smoothing curve is a target visual preference. |
| Result export | `sanitize`, `esc`, `do_export` | Update after result condition | File path, output schema, trigger condition | Do not create/overwrite result files without an explicit target contract. |
| Etterna ACC display | `ettjudge.lua`: `init`, `update`, `on_retry`, `get_level` | Init, Update, Retry | Native Ett API, setting keys, text node | Reuse optional-module/cache pattern only after confirming metric unit and profile. |

## Native bridge and probe layer

| Reusable module | Reference source | Runtime path | Project-specific dependencies | Generalization boundary |
|---|---|---|---|---|
| Native loader | Main entry: `load_native_module` | Module load / Init | DLL filename, host loader search path, ABI | Reuse only with target-specific `require` path, version/self-test, and fallback policy. |
| Native game initialization | `InitNativeState` | Init | Exact scalar argument order and source data | Treat argument order, units, and reset behavior as an ABI contract. |
| Event normalization | `NativeOnHit`, `on_hit_runtime`, `on_input_runtime` | OnHit, OnInput | Event methods and result enum | Extract host events once and pass only documented scalars. |
| Native UI snapshot | `NativeUiUpdate`, `NativeUiDirty` | Update | Fixed return order and dirty-mask meaning | Never change order/bit meaning without versioning; Lua owns Composer writes. |
| Native reset | `on_retry_runtime` plus feature resets | Retry | Native state singleton and each UI adapter | Reset Native before accepting a new event sequence; reset Lua caches/nodes too. |
| Lifecycle probe | `native_probe/malody_probe.lua` | All callbacks | Optional status node and test entry | Keep isolated from production entry; use to prove callback and loader behavior. |
| Require/self-test/replay | `native_probe/test_require.lua`, `test_native_replay.lua`, `README.md` | Standalone validation | Lua runtime, DLL architecture, expected scalar sequence | Use exact target architecture/runtime; a standalone pass does not prove host compatibility. |

## Observed native surface

The Lua code and probe documentation expose the following categories. They are interface evidence only:

```text
lifecycle:       init, reset, update, on_input, on_hit
UI snapshot:     ui_init, ui_update, ui_on_input, ui_on_judge, ui_reset
Osu adapter:     osu_config, osu_count, osu_score, osu_accuracy, osu_combo,
                  osu_max_combo, osu_last_judge, osu_last_head_judge,
                  osu_last_tail_judge
metric adapters: mm_init, mm_on_hit, mm_set_metrics, mm_value, mm_s,
                  mm_hit_count, mm_miss_count, dual_init, dual_on_hit,
                  dual_accuracy, ett_accuracy
diagnostics:     version
```

Before reusing any interface, inspect the exact module version and target code. The catalog intentionally does not assign types, result enums, default values, or algorithm semantics to undocumented members.

## Feature-order reference

The observed reference initialization order is:

```text
locale -> key overlay -> shared data -> Osu UI -> Ett UI -> Native state
-> clock -> KPS graph -> MM/S -> dual ACC -> static hides -> speed label
```

The observed update order is:

```text
Native update -> shared-data refresh -> MM native sync -> Native UI snapshot
-> Osu UI -> clock -> KPS graph -> MM/S UI -> Ett UI
```

The observed retry order clears text caches, resets Native state/UI, resets each feature, then reapplies static visibility. This is a useful model for reset completeness, but a new target must derive its own dependency order from actual source contracts.

## How to use this map

1. Select a generic module in `reusable-modules.md` based on the user's observable request.
2. Read the matching row here to find the known source family and non-generalizable dependencies.
3. Inspect the target skin for the equivalent nodes, settings, data source, and lifecycle path.
4. Adapt the generic skeleton; do not transplant reference IDs or Native calls.
5. Validate the selected feature in isolation, including retry and a missing optional-node case.
