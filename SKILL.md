---
name: malodyv-skin-engineering
description: Create, modify, diagnose, review, or explain MalodyV skins and their Lua callbacks, Composer/info.asm structure, gameplay displays, runtime timing, and Lua/native boundaries. Use when the task needs MalodyV skin contracts; do not trigger for unrelated Lua code or documentation-only typo/formatting edits.
---

# MalodyV Skin Engineering

Treat a skin as a graph: metadata-selected entry, reachable Lua modules, Composer nodes/settings, assets, callbacks, and any native dependency. Identify the affected path before changing runtime behavior; distinguish source from generated, backup, probe, and deployed copies.

## Scope and authorization

Read the whole request, not just its first verb. Explanation or review alone is read-only. “Diagnose and fix” or “audit and optimize” authorizes the corresponding source edits after investigation; a diagnosis stage does not cancel that authorization. Explicit invocation for instruction maintenance applies this skill without requiring gameplay work.

Deployment is a separate action: copy to a live skin only when requested. An already explicit destination and backup scope need no repeated confirmation. Do not commit, push, publish, or broaden the target merely because a reference describes those operations.

Preserve existing collaborators' edits. A dirty file is not itself a blocker: inspect the current content and apply a narrow patch. Stop only an operation that would overwrite unresolved conflicting work. Back up deployed or unversioned files before overwriting, retain relative paths, and record prior absence for new files.

## Work from evidence

1. Read applicable project instructions and inspect the requested artifact. For runtime work, trace metadata → entry → affected callbacks/modules/nodes. For a new skin, establish the intended graph. A documentation-only edit needs only relevant references.
2. Select the relevant reference below. Read sections on demand; do not load every manual, case study, or module example.
3. Pin result-changing facts: target version/platform/mode, callback scope, units/indexes, event identity, required nodes, state reset, and property ownership. Use existing evidence before asking for missing information.
4. Make the smallest compatible authorized change. Preserve public callback/module contracts, Composer IDs/settings/assets, event mappings, serialized data, raw observations, and existing compatibility behavior unless the requested change requires otherwise.
5. Run proportional checks using [validation](references/validation.md). Finish independent local work when the host is unavailable, then report host acceptance as unverified with an executable procedure.

Evidence labels: `official` is a published contract; `target` is inspected source/artifact; `probe` is a controlled observation; `inferred` is a hypothesis; `unknown` is unresolved. Record version/platform/mode/callback for material runtime claims. Official documents establish published API names; exact-target probes establish behavior in their scope. Changelog names do not establish signatures. Historical copies and case studies are not proof of current cross-version support.

## Read only the relevant branch

| Task | Reference and purpose |
|---|---|
| Basic callbacks or Module/Game API | [Historical basic API](references/official-basic.md); [version ledger](references/version-ledger.md) for version-sensitive capabilities |
| Free Mode notes, custom judgement or callbacks | [Free runtime supplement](references/free-mode-runtime.md), then the needed table in [historical Free API](references/official-free-mode.md) |
| Composer layout, serialized fields, conditions, triggers | [Composer components](references/composer-components.md); keep ordinary UI lookup, Note templates, Scenes and Triggers distinct |
| New skin or new Composer module conventions | [ASM creation](references/asm-creation.md); explicit user/project conventions override generator defaults |
| Legacy UIS/4.3.7 port | [Migration guide](references/legacy-uis-migration.md); linked BeatTube case is optional, target-specific evidence |
| Text, animation, locale, coordinates or UI data | [UI contracts](references/ui-data-animation.md); [Text API](references/textapi.md) for placeholders |
| Score, accuracy, grade, Full Combo or long-note state | [Gameplay rules and event phases](references/player-gameplay-rules.md) |
| Pause, rollback, offset shift or report interpretation | [Clock anomalies](references/gameplay-clock-anomalies.md) |
| Blank screen, loader, runtime files or deployment | Relevant sections of [runtime/deployment](references/runtime-deployment.md) |
| Numerical/state/cache boundary | [Implementation safety](references/implementation-safety.md) |
| Dense-section or frame-cost work | [Performance](references/performance-engineering.md); measure before claiming improvement |
| Existing or justified new native boundary | [Native architecture](references/native-architecture.md) |
| Ambiguous chart lookup | [Chart identity](references/chart-identity.md), before changing difficulty math |
| Reusable implementation pattern | Selected module in [module catalog](references/reusable-modules.md); examples require target binding |
| Historical implementation provenance | [Provenance map](references/reference-module-provenance.md) or [Basic case](references/basic-mode-case-study.md), only when needed |
| Analyzer/MinaCalc project specifically | [Analyzer conventions](references/project-conventions.md); do not impose its filenames on other skins |

## Correctness boundaries

- Do not guess undocumented APIs, callback shapes, enums, lane/index bases, timing units, note identity, wire fields or property owners. Resolve from local evidence or an isolated probe. If still unknown, pause only dependent implementation/compatibility claims and continue independent work.
- Required nodes, assets, computation and native dependencies must fail clearly if unavailable. Disable only genuinely optional features with a verified fallback. Do not use zero/default values to disguise missing gameplay data.
- For judgement/replay changes, establish an event contract and comparison oracle before changing semantics. Long notes require note identity and phase, not only lane or the latest input. Compare per-event state rather than final percentages alone.
- Each changed temporal subsystem needs a named clock and complete reset/rebase policy for derived timestamps, cursors, histories, smoothing, extrema, scales and geometry. Clamping negative `dt` alone is insufficient. Preserve raw event evidence; pause rollback is not automatically Retry. Use a non-zero regression fixture exposing the visible failure.
- Before activating native gameplay, prove load/ABI, scalar contracts and deterministic comparison. Pure implementation may be staged without host access. Deployment hashes belong to deployment verification, not prerequisites for editing a pure helper.
- Keep one continuous owner per mutable Composer property. Static ASM animation and Lua must not compete implicitly. Ordinary UI duplicate-ID lookup observations do not establish Note-template or Trigger behavior.
- For performance changes, preserve gameplay/report semantics and compare a fixed workload. A mock can measure local calls/writes and state; it cannot prove FPS or exact-host acceptance.

## Completion and handoff

Report changed files and behavior, material preserved contracts, actual checks/results, and remaining result-changing unknowns. Include source/deployed hashes and rollback scope only when those operations occurred. Keep details in an artifact when a full evidence ledger is useful.

For unavailable host checks, state target mode, fixture/action, expected visible result and required logs. Do not call a parser, mock, simulator or log-free launch gameplay acceptance. Do not stop after a plan when authorized local work remains.
