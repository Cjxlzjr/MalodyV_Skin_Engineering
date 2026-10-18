# Ambiguous Chart Identity

Read only for ambiguous external chart lookup or cache identity work.

When an external chart database lookup reports `AMBIGUOUS`, trace the active chart, the Lua/native lookup boundary, and every candidate before changing difficulty math. Match exact title, artist, creator, version, key count, and playable note count first; these fields can still describe multiple files. Do not select the first candidate or use approximate BPM, duration, or displayed level as a tie-breaker without a proved precision contract.

On the tested Windows/Steam MalodyV 6.7.12 Basic/Key host, `Game:FileHash(filename)` accepted a chart filename resolved relative to the active chart directory and returned a 32-hex-character MD5 for existing `.mc` and `.osu` files. Exact-host results matched the database's file hash, and the `Crazy Tek x0.8` difficulty prediction succeeded after this disambiguation. This is target-observed behavior, not a cross-version or cross-mode guarantee. A missing argument is invalid; capability, path behavior, return format, and failure result need a new probe on other targets. `Game:ChartInfo("cid")` was `0` for the inspected ambiguous charts, so do not assume it uniquely identifies a local file.

Use the hash only when every metadata candidate has the same filename basename. Call `Game:FileHash(common_basename)` through a protected call, validate the MD5 shape, and require exactly one database record with that hash. Then parse the selected source file and independently verify its MD5 before prediction. If basenames differ, the host call fails, the hash is absent/malformed, or the hash still matches zero or multiple records, retain `AMBIGUOUS` (or the narrower failure status) and leave prediction disabled. Record the first failure stage in the loader/lookup path, then validate with a deterministic lookup fixture and an exact-host log plus visible result.


See [version ledger](version-ledger.md) for capability history and [validation](validation.md) for evidence limits.
