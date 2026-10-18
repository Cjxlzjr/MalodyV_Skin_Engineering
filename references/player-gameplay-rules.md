# Player-Facing Gameplay Rules

Read only for player-facing score, judgement, grade, Full Combo, or long-note semantics. The Wiki 467 rules below are a historical baseline, not a verified current cross-mode contract; exact target event evidence governs implementation.

This reference summarizes the user-provided Malody Wiki page "How to play" (page 467, retrieved August 3, 2026). It exists to guide player-facing UI, labels, result panels, and acceptance tests. It is not a license to recreate engine scoring or judgement logic from display rules alone.

## Usage boundary

- Use these rules to decide which gameplay concepts a skin should display consistently: score, judgement quality, combo recovery, difficulty, grade, and Full Combo.
- Use target-proven engine data, official script APIs, or controlled runtime probes as the authoritative source for live values and final results.
- Do not infer a per-note score, the current combo bonus, a judgement enum, or a final grade by recomputing it from this document unless the target explicitly requests a verified custom implementation.
- If the target provides an existing grade, score, accuracy, or FC field, format and display that source rather than duplicating the rules below.

## Score semantics

The player-facing rules describe total score as the sum of scores earned from correctly hit notes. A note's displayed score has two components:

```text
base score + combo bonus
```

The documented base values are:

| Judgement | Base score |
|---|---:|
| Best | 1000 |
| Cool | 750 |
| Good | 400 |

The combo bonus can match the base score at full value. Therefore the documented maximum for one note is `1000 + 1000 = 2000`.

Player-facing behavior matters:

- A miss lowers the combo-bonus component.
- Sustained good performance recovers the bonus toward its normal value.
- A skin may show total score, judgement counts, combo, or a bonus/progress indicator, but it must label custom estimates as custom when they are not engine-provided.

**UI implementation rule:** do not present a static multiplier as if it were the live combo bonus unless its exact state and unit are verified. Cache formatted total-score text and update only when the visible integer changes.

## Judgement difficulty

The Wiki describes five player-facing judgement-difficulty levels. Treat their names and values as display/configuration semantics unless the exact runtime contract is verified.

| Level | Documented score bonus value | Relative normal difficulty |
|---|---:|---:|
| Amateur | 0.78 | 60% |
| Beginner | 0.85 | 80% |
| Climber | 1.00 | 100% |
| Dancer | 1.10 | 130% |
| Expert | 1.20 | 160% |

`Climber` is described as the default normal difficulty with no score bonus or penalty.

**UI implementation rule:** when adding a difficulty label, read the target's selected setting or runtime value. Do not map an arbitrary numeric value to one of these labels without verifying the target's setting enum and whether the active mode supports it.

## Result grade and Full Combo semantics

The Wiki gives these player-facing result conditions:

| Result | Documented condition |
|---|---|
| M5 | Every note score is Best. |
| M4 | No Miss and final accuracy is above 95%. |
| M3 | Final accuracy is above 90%. |
| M2 | Final accuracy is above 80%. |
| M1 | Final accuracy is above 70%. |

Full Combo is a separate non-score marker. It requires no erroneous note and no early release of a held note. It does not add score, but should be displayed distinctly on the result screen.

**Result UI rules:**

1. Show FC as a separate badge/marker, not as a numerical score bonus.
2. Do not declare FC from `combo == note_count` alone until the target confirms how hold releases and note errors are represented.
3. Use final, not live, accuracy for grade UI.
4. For thresholds described as "above", do not silently change the comparison to `>=`; verify the engine's final result source before reimplementing a grade.
5. If the engine exposes a final grade/FC field, display it directly and use this table only for labels and acceptance checks.

## Script-routing implications

Read this reference when a request involves:

```text
score, judgement counts, combo bonus, difficulty label,
result grade, score rank, Full Combo badge, or result-panel explanation
```

Combine it with:

- `official-basic.md` or exact-host evidence for API availability;
- `reusable-modules.md` for numeric display, judgement UI, and cache/reset patterns;
- `validation.md` when implementing a custom algorithm or testing final-result behavior.

## Focused acceptance checks

For a gameplay-result display feature, verify at least:

```text
score text updates from the authoritative source
judgement labels match the target's verified enum mapping
miss/reset behavior does not leave a stale combo or bonus display
difficulty label matches the selected target setting
result grade uses final accuracy/result state, not an early live value
FC is distinct from score and handles holds according to target evidence
retry clears every cached result label and badge
```

## Long-note event and accuracy rules


Treat a long note as a small event state machine, not as one repeated `OnHit` sample. The target may expose a head, zero or more body/tick/expired callbacks, a release/tail, duplicate callbacks, or a callback whose judgment enum differs from the final player-facing Miss label. Before changing any long-note score, accuracy, combo, replay, or native path:

- Build an event table for the exact target version/mode: note identity if available, lane, note kind, callback/input phase, judgment result, offset, chart time, game time, and whether the event is scoring.
- Identify the authoritative lifecycle transition for `head -> active -> tick/break -> tail/release -> completed`; do not infer it from lane alone, the most recent input event, callback order, or a display label.
- Keep per-note identity and phase separate from per-lane state. A lane is reusable and can contain overlapping or adjacent notes; a cached lane flag must not turn a tick, stale event, or second callback into a new note.
- Map every engine result explicitly, including non-scoring, expired/unjudged, break, and release results. Never assume only the visible judgment values exist, and never let an unrecognized result enter the accuracy denominator.
- Count the intended statistical unit exactly once. Decide whether the metric scores heads, tails, both, or weighted segments; make the numerator, denominator, Miss treatment, and negative-score policy explicit. A negative per-event value must not arise merely from repeated lifecycle callbacks.
- Reset and close state on release, miss/break completion, Retry, reload, replay restart, and any target-specific note identity change. Test a long note followed by another note on the same lane.

The minimum deterministic fixture is: a perfect tap, a perfect long-note head, repeated body/expired events, an early/late release, a missed head, a missed tail, adjacent same-lane long notes, duplicate callbacks, and Retry. Compare per-event count, denominator, last score, accuracy, combo, and state ownership; a final percentage alone cannot reveal duplicate-counting or stale-lane bugs.
