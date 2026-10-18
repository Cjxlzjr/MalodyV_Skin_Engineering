# Create-from-Scratch ASM Standard

Read only for a new skin or new Composer modules. Existing IDs, authorship, and target contracts remain authoritative during ordinary edits. These are generator defaults; explicit user or project conventions take precedence.

This is the default generation standard for generating a new MalodyV skin and `info.asm`. It governs files and Composer modules created by this skill. Target-host evidence still controls undocumented wire fields and runtime behavior.

### Project identity and file names

Before writing ASM, record:

```text
project_id | display_name | target_version | platform | mode | key_count | lane_order | entry_file | source_root | deployment_path
```

Apply these naming rules:

- Project IDs use lowercase ASCII `kebab-case`; Lua and asset filenames use lowercase ASCII `snake_case`.
- Convert the project ID hyphens to underscores for the entry prefix; the entry file uses `<project_snake>_<mode>.lua`, where `<mode>` is `basic`, `play`, or `free`.
- `require` strings, Composer references, and asset paths must match file casing exactly.
- Production names must be short, stable, and ASCII; do not encode coordinates, arbitrary indexes, timestamps, UUIDs, or transient implementation details in names. A lane number is allowed when it identifies a real runtime role.

### Creator and source policy

- Unless the user/project specifies another generator policy, a newly authored Composer module sets `Creator` to `muuko_skill`.
- `muuko_skill` identifies the generator/engineering skill; it does not claim original artistic authorship.
- During ordinary edits, preserve an existing third-party `Creator`; do not overwrite it unless the user explicitly requests ownership transfer.
- Do not guess unknown authorship. Keep it `unknown` in the project record and preserve the target field unless the target contract requires another value.
- Record original author, maintainer, source, and derivative status in the project handoff rather than inventing undocumented ASM metadata.

### Composer ID policy

For new ordinary UI modules, use these default ID conventions; preserve target-required IDs and scoped Note-template duplicates:

- lowercase ASCII `snake_case` only: `^[a-z][a-z0-9]*(?:_[a-z0-9]+)*$`;
- no more than 24 characters;
- one or two short semantic tokens; use three only when two cannot express the purpose clearly;
- directly describe the visible or runtime purpose;
- use a suffix only for a real role or state, such as `_bg`, `_text`, `_icon`, `_body`, or `_active`;
- be unique within its lookup scope and stable across file reordering;
- avoid UUIDs, arbitrary numeric indexes, coordinates, author names, redundant prefixes, opaque abbreviations, and implementation names. Explicit lane IDs and proven scoped template duplicates are valid when the lookup contract requires them.

Prefer `score`, `combo`, `judge`, `pause`, `hp`, `progress`, `tap_note`, `hold_body`, `hit_light`, and `lane_key`. Resolve ambiguous lookup collisions before serialization; do not rename existing or target-required IDs merely to satisfy these style defaults. Resolve collisions with the shortest semantic suffix that distinguishes the role; never append an arbitrary index.

### Mandatory Composer module record

Create one record for every new module:

```text
id:
Creator: muuko_skill
purpose:
type:
status: required | optional | unsupported
asset_dependencies:
coordinate_root:
anchor_or_pivot:
position_and_units:
layer_and_order:
lua_owner:
lifecycle_and_reset:
evidence: official | target-observed | editor-observed | probe | inferred | unknown
```

Mark every planned component `required`, `optional`, or `unsupported` for the target mode before creating its module.

### Mode-aware minimum component inventory

Build the inventory before creating modules:

| Component | Basic/Play | Free | Rule |
|---|---|---|---|
| Explicit skin entry | required | required | Entry filename and mode must be target-confirmed. |
| Background/full-screen root | required or explicitly disabled | required or explicitly disabled | Keep full-screen and play-field ownership separate. |
| Play-field root | required for gameplay modes | mode-dependent | Confirm scene transform and coordinate root. |
| Hit area | required for input gameplay | mode-dependent | Confirm lane numbering and target mode semantics. |
| Tap/Hold note templates | required for note gameplay | mode-dependent | Preserve fixed image slots when target evidence requires them. |
| Judgment feedback | required when judgments are shown | mode-dependent | Confirm result enum and callback ownership. |
| Combo/score/accuracy | required when exposed by the skin | mode-dependent | Mark unsupported rather than inventing a data source. |
| Pause/progress controls | required when exposed by the target mode | mode-dependent | Verify purpose and host restrictions. |
| HP, key feedback, masks, animation, sound, video, diagnostics | optional | optional | Add only with an identified owner, dependency, and validation case. |

### ASM/Lua/resource ownership

- Composer owns static structure, placement, assets, and static animation.
- Lua owns dynamic callbacks, runtime values, event feedback, and mutable state.
- Each mutable Composer property has one continuous owner; ASM animation and Lua updates must not compete without an explicit ownership rule.
- Keep engine adapters at the entry boundary; pure calculation modules must not directly access `Module`, `Game`, `Play`, or `Note`.
- `Init`, `Update`, `OnHit`, `OnInput`, `OnRetry`, and Note callbacks must each have a declared owner and reset policy.

### Creation checkpoints and failure routing

- **CHECKPOINT · Create contract:** resolve fields needed by the selected mode before generating dependent production bytes. Record unsupported/not-applicable fields explicitly; missing host access does not block an independently verifiable scaffold or inventory.
- **CHECKPOINT · Entry:** an unresolved or unverified entry script stops production ASM generation; do not infer it from nearby Lua files.
- **CHECKPOINT · Component:** a missing required module or asset fails creation; do not silently downgrade it to optional.
- **CHECKPOINT · Unknown wire field:** preserve unknown fields/enums as `unknown`; do not invent bytes or promote inference to contract.
- **CHECKPOINT · ID:** check new ordinary IDs against the defaults above; record target-required exceptions and verify lookup scope instead of rewriting existing contracts.
- **CHECKPOINT · Creator:** verify the chosen generator policy in newly authored modules; preserve imported authorship.

### From-scratch handoff

Use the following fields as a project record; summarize only material decisions in the final handoff:

```text
Project: project_id | display_name | source_root | deployment_path
Creator policy: new modules=muuko_skill | imported metadata=preserved
Target: version | platform | mode | key_count | lane_order
Entry: exact filename and selection evidence
Components: id | type | required/optional/unsupported | asset dependencies
Composer IDs: complete new-ID list and collision decisions
Ownership: ASM | Lua | engine for each mutable property
Evidence: official | target-observed | editor-observed | probe | inferred | unknown
Validation: static | focused | replay | exact host, with results
Backup/deployment: scope | hashes | rollback
Remaining uncertainty: only result-changing unknowns
```
