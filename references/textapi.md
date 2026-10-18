# Composer Text API Placeholders

Observed from the deployed test skin textapi Composer Text module. These are target-observed/probe results, not official stable API guarantees.

## Probe content

```text
{score}{combo}{maxcombo}{acc}{hp}{hpi}{audio}{total}{prog}{title}{art}{orgtitle}{orgart}{ver}{creator}{mod}{mode}{judge}{best}{ghit}{cool}{good}{miss}{player}{uid}{kcur}{kmax}{ktotal}{ncur}{nmax}{ntotal}{hit}{remain}{bpm}
```

## Observed mapping

| # | Placeholder | Observed meaning |
|---:|---|---|
| 1 | `{score}` | Score |
| 2 | `{combo}` | Key-mode combo in the recorded 4K probe |
| 3 | `{maxcombo}` | Highest combo |
| 4 | `{acc}` | Accuracy as a percentage with two decimal places, for example `98.53%` |
| 5 | `{hp}` | HP as a percentage with two decimal places, for example `75.00%` |
| 6 | `{hpi}` | HP as an integer in the range `0..100` |
| 7 | `{audio}` | Current music time in `MM:SS` / `00:00` format |
| 8 | `{total}` | Total time in `MM:SS` / `00:00` format |
| 9 | `{prog}` | Progress (%) |
| 10 | `{title}` | Song title |
| 11 | `{art}` | Song artist |
| 12 | `{orgtitle}` | Original song title |
| 13 | `{orgart}` | Original song artist |
| 14 | `{ver}` | Chart version |
| 15 | `{creator}` | Chart author |
| 16 | `{mod}` | Mod |
| 17 | `{mode}` | Mode |
| 18 | `{judge}` | Judgment |
| 19 | `{best}` | Best count |
| 20 | `{ghit}` | Perfect hits |
| 21 | `{cool}` | Cool count |
| 22 | `{good}` | Good count |
| 23 | `{miss}` | Miss count |
| 24 | `{player}` | Player name |
| 25 | `{uid}` | Player ID |
| 26 | `{kcur}` | Current KPS |
| 27 | `{kmax}` | Maximum KPS |
| 28 | `{ktotal}` | Total inputs |
| 29 | `{ncur}` | Current NPS |
| 30 | `{nmax}` | Maximum NPS |
| 31 | `{ntotal}` | Total valid hits |
| 32 | `{hit}` | Combo-like hit counter for non-key modes; `{combo}` is the key-mode API |
| 33 | `{remain}` | Remaining balloon hits |
| 34 | `{bpm}` | BPM |

## Lua usage

```lua
local textapi = Module:Find("textapi")
local resolved = textapi and textapi.Text or ""
```

For production, put one placeholder in each Text module and read it with `Module:Find(id).Text`; do not parse the concatenated diagnostic string.

Formatting, units, empty values, and localization remain target-dependent except where the confirmed runtime contract below specifies otherwise. Preserve opaque names such as `{ghit}` exactly.


## Reported behavior and evidence limits

These historical notes must be read together with the dated 4K record below.
Rows without a retained mode/build observation are reports to verify, not
independent proof of cross-mode support.

| Topic | Confirmed behavior | Evidence |
|---|---|---|
| `{acc}` formatting | Percentage text with exactly two decimal places, such as `98.53%` | target-observed |
| `{hp}` formatting | Percentage text with exactly two decimal places | target-observed |
| `{hpi}` range | Integer from `0` through `100` | target-observed |
| `{audio}` format | `00:00` time text | target-observed |
| `{total}` format | `00:00` time text | target-observed |
| `{combo}` scope | Key-mode combo API | target-observed |
| `{hit}` scope | Reported as a non-key-mode counter; remained literal in the dated Key probe | non-key meaning needs a scoped probe |
| Mode availability | Earlier notes claimed all-mode availability, but no per-mode probe matrix is retained here; the dated evidence below covers 4K Key only | unverified beyond recorded scope |
| Retry | Runtime values reset after Retry | target-observed |
| Pause/resume time | After pause ends, the displayed time may move backward slightly; this is a host timing behavior, not a text API failure | target-observed |
| Other values after pause | Values other than the time fields remain unchanged | target-observed |

## Exact-host probe record — August 13, 2026

Windows target host, `test` skin, 4K Key charts, independent Composer Text
nodes named `api_<placeholder>`, each containing exactly one placeholder. This
is target-observed evidence only; do not promote it to a cross-version API
guarantee.

### Read strategy

- Keep the concatenated `textapi` node only as a raw comparison artifact.
- For program logic or reports, read the final `Text` of one independent node
  with `Module:Find("api_<name>").Text`; never infer field boundaries by parsing
  the concatenated string.
- At `Init()`, dynamic fields may still be literal placeholders. In the tested
  host they resolved on the first later `Update()` (about 100 ms in the probe).
  Record initial raw text separately from final availability.
- A field is currently unavailable when its raw value remains its literal
  placeholder. Do not substitute a guessed numeric zero in a raw probe report.

### Key-mode observations

| Field group | First resolved value / final behavior | Evidence |
|---|---|---|
| Score, combo, max combo, judgments, ACC, HP, KPS, NPS, progress, BPM | Initially literal placeholders; then resolve to host-formatted text | target-observed |
| `{acc}` | Initial resolved value `100.00%`; final remains two-decimal percent | target-observed |
| `{hp}`, `{hpi}` | Initial resolved values `100.00%`, `100`; final values reflect gameplay | target-observed |
| `{audio}`, `{total}` | `00:00`-style strings; final `{audio}` equals `{total}` at completion | target-observed |
| `{prog}` | `0.00%` after initialization and `100.00%` at completion | target-observed |
| `{combo}` | Key-mode current combo; can differ from `{maxcombo}` only after a break | target-observed |
| `{hit}` | Remained literal `{hit}` throughout tested Key-mode charts | target-observed |
| `{remain}` | Remained literal `{remain}` throughout tested Key-mode charts | target-observed |
| Static metadata | `{title}`, `{art}`, `{orgtitle}`, `{orgart}`, `{ver}`, `{creator}`, `{mode}`, `{judge}`, `{player}`, `{uid}`, `{mod}` resolve at `Init()` on the tested host | target-observed |

### Probe/report interpretation

- Preserve every field as its host-formatted raw string. Do not coerce percent,
  clock, KPS/NPS, score, or integers while collecting.
- Separate `ever_unresolved` from `final_unresolved`: the first Init snapshot
  commonly contains literal placeholders for dynamic fields, which does not
  mean those fields are unsupported.
- Forced `OnHit()` and `OnInput()` captures can occur more frequently than the
  periodic 100 ms `Update()` snapshot. Store the reason for each snapshot;
  do not claim that all snapshots are 100 ms apart.
- In the tested Key charts, final `{ktotal}` and `{ntotal}` differed. Preserve
  both; do not assume they are aliases.

## Lifecycle handling

Do not assume that time is monotonically increasing across pause/resume.
Consumers of `{audio}`, `{total}`, or `{prog}` should tolerate a small backward
step after pause ends. If Lua derives deltas or rates from these values, detect
time rollback and reset the relevant accumulator instead of treating it as a
large negative gameplay event.

Retry is a reset boundary. Re-read all text nodes after Retry and clear any Lua
cache of score, combo, judgments, HP, KPS/NPS, or progress. Do not retain the
pre-Retry value merely because the Composer node still exists.

Do not infer Basic/Play/Free or non-Key availability from this 4K Key record.
Record host build, mode and callback for each newly supported field. The earlier
all-mode claim remains unverified until those observations are available.
