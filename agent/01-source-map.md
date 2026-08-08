# Source Map

## Gitmatica Packages

- `me.soullessjump.lvc`: diagnostics, user errors, player identity, and mod
  entrypoints. Do not add cross-package project/Git facades here.
- `me.soullessjump.lvc.model`: semantic data records such as `LvcManifest`,
  `LvcChunk`, coordinates, and placement DTOs.
- `me.soullessjump.lvc.storage`: versioned semantic storage, chunk codecs, hash
  indexes, object store, repository helpers.
- `me.soullessjump.lvc.capture`: world readers and capture planning/session logic.
- `me.soullessjump.lvc.diff`: reusable spatial diff grouping and source-specific
  adapters for verifier, commit, and merge data.
- `me.soullessjump.lvc.semantic`: semantic project operations and schematic builders.
- `me.soullessjump.lvc.git`: JGit branch/history/merge/remote operations.
- `me.soullessjump.lvc.task`: chunked long-running operations, recovery, restore,
  Servux/multiplayer apply, overlay tasks.
- `me.soullessjump.lvc.overlay`: tracking overlay cache, Litematica placements,
  verifier integration, origin memory.
- `me.soullessjump.lvc.gui`: Gitmatica screens, workflows, dialogs, panels.
- `me.soullessjump.lvc.gui.widgets`: custom MaLiLib widgets.
- `me.soullessjump.lvc.project`: project paths, selection storage, coordinate helpers.
- `me.soullessjump.lvc.world`: chooses authoritative world backend.
- `me.soullessjump.lvc.util`: focused helpers for NBT, GUI text fields, export files,
  loot preview.

## Litematica Reference Packages

These packages live under
`codebases/litematica/src/main/java/fi/dy/masa/litematica/`. That checkout is
reference source; standalone-addon changes belong in Gitmatica's `mixin` and
`integration/litematica` packages unless the user explicitly requests an
upstream Litematica change.

- `fi.dy.masa.litematica.gui`: existing GUI style and navigation patterns.
- `fi.dy.masa.litematica.gui.widgets`: existing list row and browser widgets.
- `fi.dy.masa.litematica.schematic.placement`: placement manager, async rebuild
  tasks, placement origin, subregions.
- `fi.dy.masa.litematica.world`: schematic world, chunks, entity lookup.
- `fi.dy.masa.litematica.schematic.verifier`: verifier lifecycle and mismatch
  data.
- `fi.dy.masa.litematica.scheduler`: task scheduling.

## Ownership Boundaries

- `LvcProjectCatalog` owns project discovery, summaries, deletion, and catalog
  caching. `LvcProject` and `LvcProjectSummary` are the canonical project DTOs.
- Semantic operations return the focused result records in `semantic`; do not
  recreate a root-level service result namespace.
- Git history, branches, remotes, merge, undo, and tree reads are called through
  their focused `git` owners. There is no `LvcProjectGitOps` facade.
- Storage codecs use `LvcContentFormat`, `LvcBinaryIO`, `LvcAtomicFiles`, and
  `LvcManifestJsonCodec` rather than embedding duplicate format/I/O logic.

## Docs Source Of Truth

- Current semantic storage: `docs/tech/lvc-semantic-storage.md`
- Conflict resolution: `docs/tech/conflict-resolution.md`
- Standalone-addon architecture: `docs/migration/architecture.md`
- Version support and backports: `docs/migration/version-support.md`
- Test strategy and commands: `docs/testing/README.md`
