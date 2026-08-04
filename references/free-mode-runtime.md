# Free Mode Runtime Supplement

Use with `official-free-mode.md`; it supplements rather than overrides documented contracts.

Evidence tags: **DOC** bundled official documentation; **LOG** official changelog name only; **PROBE** Windows MalodyV `6.7.12` runtime observation; **TARGET** callable usage recovered from a working Free Mode script. Do not generalize **PROBE/TARGET** across versions, platforms, modes, or callbacks without retesting.

## Runtime objects and callback scope

- **PROBE** `Chart`, `Note`, `Play`, and `Network` are userdata; `Play` identifies as a Free Mode judge controller. Global `Score` is `nil`; use documented `Play` setters for result state.
- **DOC** `CanPlay()` optionally filters charts; `OnProcessNote()` optionally preprocesses notes; `OnDrawNote(info)` and `OnNoteMove(info, mod, percent)` are required; `OnKey()` optionally handles raw keyboard input.
- **TARGET** A complete controller also uses `Init`, `Update`, inherited `OnInput`, and inherited `OnHit`. Obtain input/hit event objects inside their matching callbacks; do not cache them.
- Verify API presence in the callback that calls it. A method visible on an object during `OnProcessNote` may still be invalid there.

| Callback | Confirmed/recommended work |
|---|---|
| `CanPlay` | Read chart metadata; return support decision. |
| `Init` | Read Composer variables/files, clone normal modules, set miss window and static state. |
| `OnProcessNote` | Iterate chart/BPM data; call `Note:AddVirtual`. |
| `OnDrawNote(info)` | Select the current Note container and create its children. |
| `OnNoteMove(info, mod, percent)` | Update visible Note/container geometry each frame. |
| `Update` | Update ordinary UI and timeout/state cleanup. |
| `OnInput` / `OnKey` | Read `Game:InputEvent()` and implement raw-input judgment. |
| `OnHit` | Read `Game:HitEvent()` when consuming engine-produced judgments. |

## Chart and engine lists

- **DOC/PROBE** `Chart:ChartInfo(key)` returns a string. Numeric fields require `tonumber`; observed values include `key="4"`, `note="2659"`, and `bpm="170.00"`.
- **DOC/PROBE** `Chart:NoteCount()` and `Chart:BpmCount()` return numbers. `Chart:NoteAt(i)` and `Chart:BpmAt(i)` return userdata for valid indices.
- **DOC/PROBE** Chart collections are zero-based: iterate `for i = 0, count - 1`. Engine lists use `list.Length` and `list[0]`, not Lua `#list` or one-based iteration.
- **TARGET** BPM objects expose at least `time` and `bpm`; Note info commonly uses `nid`, `group`, `type`, `time`, `endtime`, `x`, `y`, `width`, `arrow`, `repeats`, and `segments` as documented.

## Virtual notes and drawing

- **DOC/TARGET** `AddVirtual` belongs to `Note`, not `Chart`, and is called only from `OnProcessNote`: `Note:AddVirtual(info)`.
- **PROBE** `Chart.AddVirtual` is `nil`; this does not indicate that virtual notes are unsupported.
- **TARGET** A virtual-note table with `type`, `time`, and `x` is accepted. One target also reads a custom field added to that table in later callbacks; treat arbitrary-field preservation as target-specific until probed.
- **DOC/TARGET** Call `Note:GetNoteModule(name[, group])` in `OnDrawNote`. The returned Note container is nonstandard and should not be assumed to support ordinary property animations.
- **TARGET** The returned container supports `root:AddSprite(name, file[, parent])` and `root:GetChild(name)`. A returned sprite supports `SetSlice(left, top, right, bottom)`.
- **PROBE** `Note.AddSprite`, `Note.AddGrid`, and `Note.GetChild` being `nil` during `OnProcessNote` is expected evidence of owner/callback scope, not evidence that container methods are absent.
- **DOC** `Note:GetGroup(name)` returns/creates a standard group container; `AddGrid` and grid `SetVertex` remain documented but were not exercised by this target.

## Movement and module properties

- **DOC/TARGET** `OnNoteMove(info, mod, percent)` runs every frame for visible Notes. `percent` is an engine list; `percent[0]` is the head progress where `0` means appearance and `1` means judgment time.
- **TARGET** Write `mod.X`, `mod.Y`, `mod.Scale`, and `mod.Alpha`; use `mod:GetChild(name)` for child-specific `Y`, `Height`, `Scale`, or `Alpha`. Created children also use `Order`.
- Preserve the target coordinate transform. Progress is not a screen coordinate and may exceed the nominal `0..1` interval before/after judgment.

## PlayMeta speed

- **TARGET** When supported by the host, `Game:PlayMeta("speed")` returns the effective play-speed multiplier for the current play session.
- Normalize with `tonumber` and treat positive values as multipliers; observed examples include `1.0`, `1.2`, `1.5`, and `0.8`, but the range is not universal.
- Use this value directly for speed-sensitive animation timing and hit-offset normalization. Do not combine it with `Game:PlayMeta("mod")` speed bits or a separate Composer Turbo correction.
- The exact callback/version contract remains target-specific; probe the call in the callback where it is consumed and use a neutral `1.0` compatibility fallback only when necessary.

## Play controller

The following are **DOC** and were visible in the **PROBE** unless noted:

| Call | Purpose / constraint |
|---|---|
| `Play:SetScore(int)` | Publish total score. |
| `Play:SetCombo(int)` | Publish current combo. |
| `Play:SetFullCombo(bool)` | Publish full-combo state. |
| `Play:SetAcc(float)` | Publish accuracy in `0..100`. |
| `Play:SetHP(float)` | Publish HP in `0..100`. |
| `Play:SetCountByType(judge, count)` | Publish counts for the four built-in result types. |
| `Play:SetFinish()` | End play and enter built-in results. |
| `Play:SetNoteFinish(nid)` | Mark a Note judged; prefer the documented `nid` argument. |
| `Play:FindNoteByTime(lower, upper)` | Return unfinished visible Notes around current time as a zero-based engine list. |
| `Play:SetMissTime(ms)` | Configure automatic miss delay. |

- **TARGET exception** One script passes full `info` to `SetNoteFinish`; this may be a permissive overload. Do not copy it unless the exact target is probed?use `info.nid` by default.
- **TARGET** Custom judgment code keeps its own score/combo/count state, then pushes it through the setters above. Do not expect a `Score` global.

## Input, hit, and general APIs

- **DOC/TARGET** `evt = Game:InputEvent()`; `evt:HitX()` returns mapped input X/lane (`-1` outside the hit area), and `evt:Type()` returns the input event enum. A target combines this with `Play:FindNoteByTime` for custom judgment.
- **DOC/TARGET** `hit = Game:HitEvent()`; `hit:JudgeResult()` returns the built-in result. Free Mode adds `hit:NoteInfoEx()` for full Note info.
- **DOC/TARGET** General calls used in Free Mode include `Game:Width()`, `Height()`, `Time()`, `AudioLength()`, and `ReadFile(name)`.
- **PROBE** `Game:Time()` returned a negative value during the pre-start countdown; do not clamp to zero unless the intended timeline explicitly requires it.
- **DOC/TARGET** `Module:Find`, `Clone`, `GetNumber`, and `GetBool` remain usable for ordinary UI/configuration. `Clone` is Init-only by documented contract.

## Post-6.2 observations

- **LOG/PROBE** `Game:PlayMeta` exists. On `6.7.12`, `PlayMeta("judge")` returned number `4`, `PlayMeta("mod")` returned number `0` with no Mod, and an unknown key also returned `0`; non-`nil` does not prove key validity.
- **TARGET** `PlayMeta("speed")` is used as a flow-speed input. Its type, units, and values under speed Mods remain unconfirmed.
- **LOG/PROBE** `Game:FileHash` is callable, but two unverified names returned `""`; path scope, missing-file behavior, algorithm, and output format remain unknown.
- **LOG/PROBE** `Network:DoRequest` is a Free Mode function. `Network.OnResponse` is `nil`; treat `OnResponse(...)` as a global callback name pending confirmation of parameters and trigger behavior.

## Minimal original example

This example is illustrative, not a recovered target implementation; replace names/assets and probe the exact version.

```lua
local FX_TYPE = 101

function CanPlay()
    return tonumber(Chart:ChartInfo("key")) == 4
end

function OnProcessNote()
    Note:AddVirtual({ type = FX_TYPE, time = 1000, x = 1 })
end

function OnDrawNote(info)
    local root = Note:GetNoteModule(info.type == FX_TYPE and "effect" or "normal")
    root:AddSprite("body", info.type == FX_TYPE and "effect.png" or "note.png")
end

function OnNoteMove(info, mod, percent)
    local progress = percent[0]
    mod.Y = -540 + 1080 * progress
    if progress < 0 or progress > 1.1 then
        mod.Alpha = 0
    else
        mod.Alpha = 100
    end
end
```
