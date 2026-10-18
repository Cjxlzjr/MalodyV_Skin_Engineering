# BeatTube Migration Case Study

Read only when comparing against this specific migration. Return to the [migration guide](legacy-uis-migration.md) for general inventory, units, and acceptance.

The following is a concrete standard-key example, not a universal MalodyV
contract. Paths, asset names, numeric values, and failed experiments are
`target` or `probe` evidence from this case and must be remeasured for another
skin, key count, platform, or host version.

### Legacy side

The source root was:

```text
D:\codex\1173_BeatTube_Skin
```

The inspected files were `base.mui` and:

```text
script-key-4K.mui
script-key-5K.mui
script-key-6K.mui
script-key-7K.mui
script-key-8K.mui
```

The 4K `note-[1-4]` default example was:

```text
pos=16.7%$22.2%,100%+50
pos2=16.7%$22.2%,110
size=100,100
```

The 4K conditional branch used a different position and size:

```text
pos=50%-240$160,100%+50
pos2=50%-240$160,110
size=95,95
```

The 5K–8K examples used `size=96.73,120` and a conditional
`size=79.66,98.84`. These are source observations, not portable target values.

The legacy manual used in this case defined approximately:

```text
1 legacy relative unit = window height / 720
```

The conversion must still be checked against the exact legacy version and
platform before being reused.

### MalodyV side

The target skin root was:

```text
D:\Program\Steam\steamapps\common\MalodyV\skin\beattube
```

The inspected metadata file was:

```text
D:\Program\Steam\steamapps\common\MalodyV\skin\beattube\info.asm
```

The observed native note categories were:

| Target artifact | Observed role | Migration meaning |
|---|---|---|
| `Type=7` | ordinary/tap note | Native note rendering; verify whether position fields are consumed. |
| `Type=6` | long-note/hold note | Native hold rendering; verify head/body/tail resource ownership. |
| `Source=1` | lane/source distinction in this case | Do not assume the same source index in another mode. |
| `noten.png` | ordinary note art | Adjust visible alpha bounds only if native position is fixed. |
| `noteh.png` | alternate ordinary note art | Keep canvas and anchor consistent with the target template. |
| `notet.png` | hold tail/related art | Validate together with the hold template. |
| `lnmid.png` | hold middle art | Check width, tiling/stretching, and alignment separately. |

An experiment added negative `DX` to lanes 1–2 and positive `DX` to lanes 3–4.
The file saved successfully, but the exact-host image did not move and there was
no evidence that the game reset the values to zero. The scoped conclusion is
that these native `Type=6/7` note instances did not consume the generic offset
fields for per-lane visual movement. This is not proof that every MalodyV mode
ignores `DX`; it is a target/probe result for this BeatTube path.

### Visual-size and scene-scale observations

The BeatTube target showed approximately:

```text
playfield width ≈ 1680 × SceneScale
4K lane width   ≈ 420 × SceneScale
```

For a tablet reference height of `1908`, the case-study starting estimate was:

```text
SceneScale ≈ 1.0167 × target screen height / 1908
```

This produced approximate starting values of `0.58` at `1920×1080` and `0.60`
at `1800×1125`. These values are target/probe evidence and must not be copied
to another mode without a new lane/playfield measurement.

Because scene scale changes both playfield geometry and native artwork, the
remaining Note-size correction used transparent padding inside the target PNG
canvas. This changes the visible alpha bounding box while preserving the
native instance and judgment location. Ordinary and hold resources were tuned
separately; changing one PNG is not sufficient to prove hold equivalence.

### Judgment-position mapping

Two target calibration points were recorded:

```text
MalodyV judgment position -160 → Y-axis 0u
MalodyV judgment position 1500 → Y-axis 970u
```

The case-study linear mapping was:

```text
Y(u) ≈ (judgment position + 160) × 0.584337
judgment position ≈ -160 + 1.711340 × Y(u)
```

This mapping is valid only for the recorded BeatTube coordinate definition and
test range. A different skin or mode must establish its own calibration points.
