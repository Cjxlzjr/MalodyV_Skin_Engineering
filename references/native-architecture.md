# Lua and Native Architecture

Use this reference only when measured or clearly dominant pure computation justifies a Lua/native boundary. Keep ordinary Composer lookup, formatting, animation, and small event logic in Lua.

## Contents

- [Responsibility Boundary](#responsibility-boundary)
- [Load and ABI Probe](#load-and-abi-probe)
- [Scalar Event Contract](#scalar-event-contract)
- [Lifecycle and UI Snapshot](#lifecycle-and-ui-snapshot)
- [Replay and Validation](#replay-and-validation)
- [Deployment and Rollback](#deployment-and-rollback)

## Responsibility Boundary

Keep host ownership in Lua:

```text
Lua
- Malody lifecycle callbacks
- Game, Module, and other host API calls
- event extraction and normalization
- Composer node lookup and property writes
- resources, animation, files, JSON, and diagnostics UI

Native module
- pure mathematics and deterministic algorithms
- bounded counters, state machines, histories, and numerical animation state
- fixed scalar inputs and fixed multi-return snapshots
```

Move code only when it is independent of Malody userdata and testable from scalar inputs. Keep the hot gameplay ABI scalar: never pass `Game`, `Module`, Composer nodes, host userdata, closures, C++ objects, STL containers, exceptions, or borrowed pointers across it. A Lua table returned by a diagnostic binding such as `abi_info()` is allowed through the Lua C API; it is not a C struct ABI or a per-frame snapshot design.

Prefer a C ABI. If C++ is required, export `extern "C"` functions and keep every C++ type behind the DLL boundary.

## Load and ABI Probe

Prove loading before enabling migrated gameplay on the target. Pure implementation and deterministic comparison may be staged while host access is unavailable:

1. Confirm target host version, Lua version, platform, architecture, mode, and callback.
2. Add the skin directory to `package.cpath` only when a target probe proves it is required and supported.
3. Require one isolated module such as `example_native`.
4. Export exactly `luaopen_example_native` with unmangled C linkage.
5. Resolve Lua API symbols from the runtime already loaded by the host; never load a second Lua runtime.
6. Check every required symbol and fail clearly when initialization is incomplete.
7. Use a minimal probe surface (`version`, `abi_info`, `self_test`) to isolate loader failures; do not enable production gameplay until standalone and exact-host probes pass.

Minimal project shape:

```text
example_probe/
|- CMakeLists.txt
|- src/example_native.c
|- bin/
`- test_require.lua
```

Minimal CMake shape for Windows x64 and MinGW-w64:

```cmake
cmake_minimum_required(VERSION 3.20)
project(example_native C)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_library(example_native SHARED src/example_native.c)
set_target_properties(example_native PROPERTIES
    PREFIX ""
    OUTPUT_NAME "example_native"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin"
)
```

Build into staging:

```powershell
cmake -S <probe-directory> -B <build-directory> -G "MinGW Makefiles"
cmake --build <build-directory> --config Release
```

When headers or an import library are unavailable, declare and resolve only the exact Lua functions used. Verify signatures, calling convention, integer/number types, stack behavior, and constants against the target ABI; a guessed declaration can crash the host.

Lua-side load probe:

```lua
package.cpath = table.concat({
    "<skin-directory>/?.dll",
    package.cpath,
}, ";")

local native_module = require("example_native")
assert(_VERSION == "Lua 5.3", "unexpected Lua version: " .. tostring(_VERSION))
assert(type(native_module.version()) == "string")
assert(type(native_module.abi_info()) == "table")
assert(native_module.self_test() == true)
```

Inspect the binary architecture, dependencies, and export table before host launch. The export must be the exact `luaopen_<module>` symbol.

## Scalar Event Contract

Extract a host event once in Lua and pass stable scalar fields:

```text
result
offset_ms
object_type
lane
input_type
event_time_ms
```

- Use explicit units such as milliseconds and document every conversion.
- Preserve offset sign, host enums, lane indexes, and object identity unless the boundary documents a conversion.
- Validate malformed fields or apply an explicit fallback before calling native code.
- Feed the same normalized snapshot to every subsystem; do not reinterpret the host event independently.
- Reset on retry, chart reload, mode transition, and detected time rollback.

During development, log normalized scalars and sequence numbers rather than host userdata.

## Lifecycle and UI Snapshot

Use a small deterministic surface:

```lua
native_module.initialize(...)
native_module.reset()
native_module.update(time_ms)
native_module.on_input(input_type, time_ms)
native_module.on_event(result, offset_ms, object_type, lane, input_type)
```

For UI-related numerical state, expose a separate snapshot path:

```lua
native_module.ui_initialize(...)
native_module.ui_update(time_ms)
native_module.ui_on_event(...)
native_module.ui_reset()
```

Document the type and order of every return value. Treat a fixed return order as an ABI contract; add a versioned function or new snapshot rather than inserting values into an existing return list.

Keep Composer references in Lua and apply only dirty groups:

```lua
local now_ms = Game:Time()
native_module.update(now_ms)

local dirty_mask,
      display_value,
      display_alpha,
      display_scale = native_module.ui_update(now_ms)

if dirty_mask & 1 ~= 0 then
    display_node.Text = tostring(display_value)
end
```

When the host lacks native bitwise operators, use its documented bit library or a verified compatibility helper. Prefer direct multiple returns over allocating a Lua table every frame.

Keep histories and bucket math in native code only when profiling justifies it. Composer graph nodes still update in Lua, and only when a bucket, viewport, peak, or visible result changes.

Illustrative adapter lifecycle (the `...` placeholders inside non-vararg callbacks require concrete arguments before this is runnable Lua):

```lua
function Init()
    native_module = require("example_native")
    assert(native_module.self_test())
    native_module.initialize(...)
    native_module.ui_initialize(...)
    find_required_nodes()
end

function Update()
    local now_ms = Game:Time()
    native_module.update(now_ms)
    apply_native_snapshot(native_module.ui_update(now_ms))
end

function OnHit()
    local event = Game:HitEvent()
    if event then
        local result, offset_ms, object_type, lane, input_type = normalize_event(event)
        native_module.on_event(result, offset_ms, object_type, lane, input_type)
        native_module.ui_on_event(result, offset_ms, input_type)
    end
end

function OnRetry()
    native_module.reset()
    native_module.ui_reset()
    reset_lua_owned_ui()
end
```

Make initialization idempotent or reject duplicates explicitly. Clear state before accepting events after a chart or mode transition.

## Replay and Validation

Validate in increasing scope:

1. Parse affected Lua files and changed entry/dependency paths with the target Lua toolchain.
2. Run standalone `require`, `abi_info`, and `self_test`.
3. Inspect DLL architecture, dependencies, and exports.
4. Replay normalized scalar events against the Lua/reference implementation.
5. Exercise initialization, update, input, event, retry, reload, and time rollback.
6. Test sparse and dense input plus sustained or multi-part objects relevant to the mode.
7. Confirm counters, histories, dirty masks, and cached snapshots reset deterministically.
8. Compare node-write count and frame cost before and after a performance migration.
9. Run an isolated exact-host probe before a full skin session.

For a zero-difference migration, compare every event instead of only the final result. Include sequence number, normalized fields, cumulative counters, accuracy, state hash, history values, and reset state. Stop at the first mismatch and print both snapshots.

Keep a Lua reference implementation in a test-only path. Use a temporary shadow mode or feature switch to compare outputs without changing UI ownership; preserve a clear rollback path until replay and host tests pass.

Classify failures:

```text
DLL not found                -> filename, search path, package.cpath
module not found             -> require name or loader registration
architecture mismatch        -> host, Lua, compiler, or DLL bitness
export missing               -> export table, C linkage, calling convention
host Lua API unavailable     -> wrong runtime or unsupported symbol
callback not entered         -> mode, entry, or lifecycle contract
UI node missing              -> Composer or deployment mismatch
retry not cleared            -> incomplete native/Lua reset
event values differ          -> unit, sign, indexing, or normalization
host crash                   -> ABI, stack, lifetime, or invalid pointer
```

Expose switchable, rate-limited diagnostics for module version, ABI assumptions, initialization state, last event sequence, and reset count. Do not log every frame in release behavior.

## Deployment and Rollback

Build into staging, test there, then deploy only the tested DLL and required Lua files. Before host launch, verify:

- exact target directory and metadata-selected entry;
- architecture and dependent runtime DLLs;
- module filename, `package.cpath`, and export name;
- resolved host Lua symbols;
- source and deployed file hashes;
- previous known-good DLL/Lua snapshot and rollback instructions.

Keep an unvalidated native migration out of the production path until the load probe and host smoke test pass. An already-required native dependency does not become optional because a probe is unavailable. If a fallback exists, make it explicit and prevent stale values. If no safe fallback exists, fail clearly before gameplay behavior depends on partial native state.

At the Lua boundary, validate scalar arguments, cap counts/history/string sizes, avoid unbounded callback allocation, clear state on every reset path, and keep native state single-instance unless explicit handles are designed.

A complete native handoff states target host/version, architecture, required files, load path, lifecycle assumptions, scalar units and indexes, reset rules, validation commands and results, unsupported modes, and rollback procedure; include deployed hashes only if deployment occurred.
