# Basic-Mode Case Study

Scope: historical Lua implementation snapshot, not the current active graph. No source hash or exact host build is recorded here. Recheck the target before adopting callback order, speed compensation, or lane-keyed state; the native-backed provenance map describes a different snapshot.

Use this as a reusable architecture case, not a universal API contract. Sources inspected: the target Basic Mode entry, supporting judgment modules, and packaged skin metadata.

## Deployment snapshot

- Packaged entry: the target Basic Mode entry script; Free Mode is off. Observed Composer field values: track angle `45`, scene scale `0.37`, judgment position `260`; treat them as this skin's settings, never defaults.
- Entry loads `require "OsuJudge"` and `require "EttJudge"`; supplied files are `OsuJudge.lua` and lowercase `ettjudge.lua`. Windows may tolerate case differences, but portable deployment must match exact case.
- UTF-8 files have no BOM. The entry is a Basic/Play lifecycle adapter; it does not implement Free Mode note callbacks.

## Module architecture and lifecycle

- The entry script owns engine callbacks and dispatches to subsystem state tables: `KEYOVERLAY`, `KPS_CHART`, `CLOCK`, `MM`, `DUAL_ACC`, static hiding, shared data, `OsuJudge`, and `EttJudge`.
- `ettjudge.lua` / `OsuJudge.lua` use `local M={}` plus `M.init/update/on_hit/on_input/on_retry/reset_state`, with private `local function` helpers and `return M`.
- Historical dispatch order in this snapshot (recheck dependencies on a newer target): `Init` initializes overlay, osu, Etterna, shared data, clock, KPS, MM, dual ACC, then static hiding; `Update` runs osu before shared/UI updates; `OnHit` runs osu, dual ACC, then Etterna; `OnInput` runs overlay, osu, Etterna, then KPS; `OnRetry` resets every subsystem and reapplies static hiding.

## Reusable coding patterns

- Alias hot standard-library functions at file scope (`math.floor`, `string.format`, etc.).
- Group mutable state by feature; prefix helpers with the feature name and separate config lookup, module lookup, cloning, initialization, update, input, visibility, and reset.
- Cache `Module:Find` results during initialization. Clone templates only during initialization; use `Module:Shadow` for bounded transient trails.
- Preallocate reusable animation tables and numeric/string caches to reduce callback allocations.
- Use ring buffers and bounded scans for recent input; use cached “last displayed” values to avoid redundant `Text` writes.
- Preserve original alpha/position before hiding; support optional modules with `nil` checks.
- This older snapshot combined Turbo and speed Mod. Reuse that fallback only if its target semantics are proved; prefer a separately validated effective-speed API, and never combine both transformations.
- Reset counters, lane state, cursors, animation visibility, smoothing targets, export flags, and caches explicitly in `OnRetry`.

## Feature inventory

| Feature | Composer config | Module/template names and dynamic names |
|---|---|---|
| Static hide | none | `当前分辨率`, `点击进入修改`, `kpsbar` |
| Key overlay | `按键阴影开关`, `按键动画与计数器开关`, `按键阴影持续时间(ms)`, `Turbo倍速修正 (50-200)` | `overlay1..8`, `overlayBar`, `keyImage_1`, `key_Count`; clones `overlays_i`, `keyImage_2_custom_i`, `keyImage_1_i`, `counts_i` |
| KPS chart | `KPS条开关`, `KPS显示模式(关:折线图 开:直方图)`, Turbo | `kpsbarbg`, `kpsbar` or `kpsline`, `max5text`, `maxtext`, `当前分辨率`; clones numbered bars/lines |
| Clock | `时钟开关` | `clock`, `clock_bar_1`; clones `clock_bar_i` |
| MM | `MM值显示开关` | `mm_value`, `s_value`, `ACC`, `SCORE`, `NTOTAL`, `miss_count`, `malody_mod_info` |
| Dual ACC | `双判定ACC显示开关`, `开启 ProJudge (关闭为 Stb)`, `自定义判定1 (1A-5E)`, `自定义判定2 (1A-5E)`, `加减速Mod影响开关`, Turbo | `acc_target1_text`, `acc_target2_text`, `JUDGE`, `malody_mod_info` |
| Etterna ACC | `开启 Etterna ACC`, `Etterna 判定等级 (4-9)`, Turbo | `ett_acc` |
| osu! judge | `OD(支持1位小数)`, `Osu谱面使用.osu文件中的OD值`, `Score v2`, `开启Hard Rock(HR)`, `尾判`, `Osu判定显示`, `Osu连击显示`, `Offset残留时间(ms)`, `游玩时自动隐藏设置信息`, Turbo | `indicator_osu`, `m_off1..4`, `judge_rainbow`, `judge_yellow`, `osu_score`, `osu_acc`, `osu_ratio`, `osu_combo`, `osu_combo1..4`, `osu_judge_perfect`, `osu_judge_great`, `osu_judge_good`, `osu_judge_ok`, `osu_judge_meh`, `osu_judge_miss`, `osu_judge_count1..6`, Malody/player/result fields, window/info labels |

Exact osu/Malody support names also include `malody_combo`, `malody_combo1..4`, `malody_score_value`, `malody_acc_value`, `player_id`, `player_name`, `best_count`, `cool_count`, `good_count`, `miss_count`, `max_combo`, `osu_perfect_window`, `osu_great_window`, `osu_good_window`, `osu_ok_window`, `osu_meh_window`, `osu_miss_window`, `osu_info_display1..3`, `judgement_displayed`, and `combo_displayed`.

## Actual API and property surface

- Engine/module calls: `Module:Find`, `Module:GetBool`, `Module:GetNumber`, `Module:GetString`, `Module:Clone`, `Module:Shadow`, `Game:StartTime`, `Game:Time`, `Game:AudioLength`, `Game:Width`, `Game:ChartInfo`, `Game:HitEvent`, `Game:InputEvent`, `Game:PlayMeta`, `Game:ReadFile`, `Game:WriteFile`.
- Event calls: `JudgeResult`, `Offset`, `HitX`, `NoteType`, input `Type`.
- Written properties: `X`, `Y`, `Width`, `Height`, `Rotate`, `Alpha`, `Text`, `Value`; animations: `DoMoveX`, `DoMoveY`, `DoResize`, `DoHeight`, `DoAlpha`.
- `ChartInfo` numbers are usually wrapped in `tonumber`, but one key-count path relies on implicit conversion; new code must normalize explicitly.
- osu export sanitizes Windows filenames, escapes JSON strings, reads optional `.osu` OD, and writes a result JSON after audio completion time.

## Speed metadata semantics

- A skin can use `Game:PlayMeta("speed")` as the single source of effective play speed when the host supports the key.
- Do not combine a valid API value with separate Composer Turbo correction or Mod-text/bit parsing; applying both causes double scaling of speed-sensitive timing or `Game:HitEvent():Offset()`.
- Keep the API value as a positive multiplier and use a neutral `1.0` only for compatibility when the API call is unavailable.

## Judgment and state semantics

- Both judge modules transform `Game:HitEvent():Offset()` using configured Turbo and observed speed-Mod factors.
- Etterna maps judgment level 4..9 to timing scale, tracks long-note state by `HitX`, scores the head once, and ignores tail scoring.
- osu logic maintains per-lane long-note state, distinguishes head/tail through the most recent input type, supports score v1/v2, HR windows, optional tail judgment, smoothed ACC/combo displays, offset trails, and delayed export.
- Judgment enums `1..4` and combo break `8` are target assumptions backed by the bundled official mapping; re-check when adding modes or callbacks.

## Do not generalize blindly

- Exact Composer names, 4K-oriented formulas, maximum 16 input indices, Mod bit masks `16/32/256`, speed labels, score formulas, and lane-keyed hold state are skin-specific.
- `X=-5000` and `Alpha=0` are pragmatic hiding strategies, not equivalent contracts; off-screen modules may still animate or consume work.
- Sentinel `5210721`, hard-coded dimensions/counts, text-parsed configuration, and large single-file state increase maintenance risk.
- Top-level unprotected `require`, global `InitSharedData` / `UpdateSharedData`, direct access to optional module fields, and absence of capability/version gates can blank the skin on unsupported hosts.
- Preserve successful algorithms when fixing UI or lifecycle issues; isolate risky/optional exports, file access, and judge conversions behind validation and concise diagnostics.

Lane-keyed hold state and the latest input event are historical implementation details, not sufficient note identity. For new or repaired long-note logic use the [event/phase contract](player-gameplay-rules.md#long-note-event-and-accuracy-rules).
