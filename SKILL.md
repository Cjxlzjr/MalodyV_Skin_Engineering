---
name: malodyv-lua
description: Trace and engineer MalodyV Lua skin graphs. Use when creating, modifying, debugging, reviewing, optimizing, or explaining a skin; diagnosing load or blank-screen failures; or changing Composer, Play/Free Mode callbacks, gameplay displays, replay behavior, or Lua/native boundaries.
---

# MalodyV Lua Skin Engineering

Treat a skin as a **skin graph**: the metadata-selected entry, reachable Lua modules, Composer nodes and settings, assets, engine callbacks, deployment copies, and optional native modules that jointly produce the behavior.

## Core Loop

Run this loop for every task. Each step has a completion criterion; do not advance until it is met.

1. **Trace the graph.** Find the active skin root, metadata-selected entry, reachable `require` edges, relevant callbacks, mode, host/version clues, Composer dependencies, assets, deployment copies, and available probes. Complete when the requested behavior has one identified execution path or the ambiguity is explicitly blocking.
2. **Route the branch.** Select the smallest branch in [Branch Router](#branch-router) and read every referenced file named by that row. Complete when all API, deployment, gameplay, or native facts needed for the edit have a source.
3. **Pin the contracts.** Record engine-facing assumptions as `official`, `target-observed`, `probe-observed`, `inferred`, or `unknown`. Complete when every changed callback, field, unit, index, Composer ID, setting key, reset path, and public module surface is accounted for.
4. **Patch the narrow path.** Preserve the target architecture and public contracts; change only the reachable path needed for the request. Complete when the user-visible behavior is implemented without an unrelated rewrite.
5. **Prove the behavior.** Climb the validation ladder from static checks to focused behavior and exact-host evidence. Complete only at the task-class gate in [Completion Gates](#completion-gates).
6. **Hand off the evidence.** Report changed files, preserved contracts, checks run, host verification status, remaining uncertainty, rollback scope, and the exact in-game observation that confirms success.

Use this compact ledger while working:

```text
fact | source: official/target/probe/inferred/unknown | confidence | validation needed | decision
```

Never promote an inferred or unknown fact into an engine contract without saying so.

## Branch Router

Load only the row that matches the requested behavior. A task may require more than one row, but each added reference must earn its context cost.

| Request touches | Read before editing | Required evidence |
|---|---|---|
| Existing UI text, score, accuracy, combo, clock, KPS, layout, or animation | [`official-basic.md`](references/official-basic.md), [`ui-data-animation.md`](references/ui-data-animation.md); [`reusable-modules.md`](references/reusable-modules.md) when creating a common feature | Authoritative value, node/property owner, update cadence, reset behavior, and visible acceptance condition |
| Composer settings, IDs, templates, clones, anchors, layers, masks, or localization | [`composer-components.md`](references/composer-components.md), [`official-basic.md`](references/official-basic.md); configuration/locale adapter in [`reusable-modules.md`](references/reusable-modules.md) | Stable keys and IDs, required/optional classification, fallback behavior, and representative config/locale smoke test |
| Play Mode lifecycle, input, hit events, score, retry, replay, or chart parsing | [`official-basic.md`](references/official-basic.md), [`implementation-safety.md`](references/implementation-safety.md), [`validation.md`](references/validation.md); [`player-gameplay-rules.md`](references/player-gameplay-rules.md) for player-facing semantics | Callback ownership, event shape, units/indexes, retry/reload behavior, and focused host or replay evidence |
| Free Mode, virtual/custom notes, grids, note judgment, or version-sensitive callbacks | [`official-free-mode.md`](references/official-free-mode.md), [`free-mode-runtime.md`](references/free-mode-runtime.md), [`version-ledger.md`](references/version-ledger.md), [`validation.md`](references/validation.md) | Exact mode, callback and call shape, target version, sparse/dense cases, and hold/chord behavior |
| Native bridge, DLL, ABI, hot path, or performance migration | [`native-architecture.md`](references/native-architecture.md), [`runtime-deployment.md`](references/runtime-deployment.md), [`implementation-safety.md`](references/implementation-safety.md), [`validation.md`](references/validation.md) | Isolated load probe, scalar boundary, ABI/export/architecture evidence, replay comparison, deployment hash, and rollback path |
| Difficulty, statistics, analyzer, or chart-wide algorithms | [`implementation-safety.md`](references/implementation-safety.md), [`validation.md`](references/validation.md); relevant catalog entry in [`reusable-modules.md`](references/reusable-modules.md) | Chart-wide versus callback boundary, deterministic fixtures, malformed/empty input behavior, and measured cost where optimization is claimed |
| Blank screen, load failure, missing module, encoding, or deployment mismatch | [`runtime-deployment.md`](references/runtime-deployment.md), [`project-conventions.md`](references/project-conventions.md) when repository-specific | Active copy, entry and loader path, encoding, last successful lifecycle stage, exact error, and safe rollback |
| A large Basic Mode implementation or a documented local reference module | [`basic-mode-case-study.md`](references/basic-mode-case-study.md) or [`reference-module-provenance.md`](references/reference-module-provenance.md) only when that provenance is actually relevant | Portable pattern separated from target-specific IDs, settings, assets, ABI names, and assumptions |

## Trace the Active Graph

Inspect before asking the user for implementation details:

1. Read applicable `AGENTS.md`, project context, metadata, Lua entry files, Composer artifacts, and repository conventions.
2. Distinguish source, generated, backup, `.buffer`, probe, obfuscated, and deployed copies.
3. Search lifecycle definitions, `require` edges, `Module:*`, `Game:*`, event access, Composer IDs, settings, and assets on the requested path.
4. Classify the target as resource-only, single-file Lua, modular Lua, configuration-driven, analyzer/tooling, probe/test, native-assisted, generated/obfuscated, or legacy/experimental.
5. Reuse the closest working target pattern without normalizing every skin to one architecture.

Derive this brief before mutation:

```text
user-visible goal
active skin root and entry
mode and reachable callbacks
affected nodes/assets/settings
authoritative data and units
state, reset, and property ownership
files that must change
observable acceptance behavior
unresolved assumptions
```

Ask only when local evidence cannot distinguish an active entry, node, mode/API contract, destructive deployment target, or conflicting required behavior. Infer low-impact style, naming, cache, formatting, and animation details from the target and report those assumptions afterward.

## Contract Rules

Preserve these contracts unless the request explicitly changes them:

- Metadata-selected entry, public callbacks, module names and returns.
- Composer IDs, setting keys, templates, clone names, assets, and serialized data.
- Event fields, judgment mappings, index bases, units, masks, modifiers, hold/chord semantics, and result fields.
- Existing compatibility behavior, locale keys, reset semantics, and deployment shape.

Use exact runtime observation on the target version/mode/callback first, active target code second, bundled official references third, and general Lua knowledge last. A fact observed in one callback or mode applies only to that boundary.

Prefer a local helper for a small isolated behavior. Add a module when the target already uses modules or the feature owns an independent lifecycle/state boundary. Introduce native code only for measured or clearly dominant pure computation that can be expressed through stable scalar data.

Keep required initialization failures visible. Isolate optional UI, diagnostics, or analysis failures only when a real fallback exists.

## Mutation Safety

Before overwriting a deployed or unversioned skin file, create a fresh timestamped backup that preserves relative paths and records whether rollback covers source, deployment, or both. For a new destination file, record its prior absence so rollback can remove it.

Do not deploy merely because the user mentioned a game directory. Validate the source first; deploy only when requested, back up the destination immediately before replacement, copy only confirmed files, and verify deployed hashes.

## Validation Ladder

Validate in increasing scope and stop claiming only what the completed rung proves:

1. **Static:** encoding/no-BOM where required, Lua syntax, reachable modules, exported helper surfaces, required Composer lookups, and obvious nil/unit/index errors.
2. **Focused behavior:** lifecycle order, event/value path, reset/retry/reload/time-rollback behavior, missing optional nodes/settings, edge values, and property ownership.
3. **Replay or deterministic probe:** normalized inputs, per-event snapshots, first mismatch, state hash/counters, and zero-difference comparison for gameplay or algorithm migrations.
4. **Exact host:** correct version, platform, mode, active copy, callback entry, visible result, relevant log stage, and native/deployment evidence when applicable.

Parsing proves syntax, not visual or gameplay correctness. When the host is unavailable, finish every reachable lower rung and give the exact manual action, mode, expected screen behavior, and log evidence needed to close the gap.

## Completion Gates

- **Simple UI/data task:** active entry and node confirmed; one compatible path changed; static checks pass; authoritative value, edge value, missing-node, and reset behavior are checked; exact in-game observation is stated.
- **Moderate lifecycle/configuration task:** reachable graph and Composer/config contract recorded; initialization plus retry/reload/fallback paths checked; focused host smoke test passes or a precise unavailable-host procedure is supplied.
- **High-risk gameplay/native/migration task:** exact event/state contract is evidence-backed; reference and new paths are replay-compared; ABI/deployment and rollback are verified where applicable; no judgment, timing, indexing, reset, or compatibility assumption remains silently unresolved.

## Handoff

Keep the report compact and include:

```text
Completed: changed files and user-visible behavior
Skin graph: entry -> reachable modules -> affected callbacks/nodes
Preserved contracts: public API, Composer, units, mappings, compatibility
Evidence: important official/target/probe facts and remaining inference
Validation: commands/checks, host availability, and results
Acceptance: exact mode, action, visible behavior, and relevant log evidence
Backup/deployment: snapshot scope, destination, hashes, and rollback
Remaining uncertainty: only items that can still change the result
```

## Reference Map

- [`ui-data-animation.md`](references/ui-data-animation.md): data displays, animation ownership, coordinates, speed handling, and visual acceptance.
- [`native-architecture.md`](references/native-architecture.md): Lua/native boundary, probe, ABI, lifecycle, replay, deployment, and rollback.
- [`reusable-modules.md`](references/reusable-modules.md): parameterized lifecycle, config/locale, display, animation, overlay, KPS, clock, metric, judgment, native, probe, and replay modules.
- [`reference-module-provenance.md`](references/reference-module-provenance.md): portable patterns versus private/reference-only dependencies.
- [`official-basic.md`](references/official-basic.md): historical Basic/Composer API baseline.
- [`official-free-mode.md`](references/official-free-mode.md): historical Free Mode API baseline.
- [`free-mode-runtime.md`](references/free-mode-runtime.md): target-observed Free Mode runtime behavior.
- [`runtime-deployment.md`](references/runtime-deployment.md): host probes, loaders, encoding, lifecycle ownership, and load failures.
- [`implementation-safety.md`](references/implementation-safety.md): engine boundaries, normalization, architecture, and performance.
- [`validation.md`](references/validation.md): static, behavioral, replay, algorithm, and failure checks.
- [`composer-components.md`](references/composer-components.md): settings, module types, placement, layers, masks, and variables.
- [`version-ledger.md`](references/version-ledger.md): version gates, changelog evidence, and required probes.
- [`player-gameplay-rules.md`](references/player-gameplay-rules.md): player-facing score, judgment, grade, and Full Combo semantics.
- [`basic-mode-case-study.md`](references/basic-mode-case-study.md): target-specific Basic Mode case study.
- [`project-conventions.md`](references/project-conventions.md): optional repository-specific architecture and naming.
