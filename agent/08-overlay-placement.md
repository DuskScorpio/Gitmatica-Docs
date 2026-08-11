# Overlay Placement

`LvcTrackingOverlayService` orchestrates Gitmatica tracking overlays.

Supporting ownership is intentionally split:

- `LvcTrackingOverlayRevisionResolver`: current/parent/root-air revision choice.
- `LvcTrackingOverlayDescriptorStore`: descriptor serialization and atomic I/O.
- `LvcTrackingOverlayRegistry`: active overlay and remembered-origin lifetime.
- `LvcTrackingOverlayEntry`: cache entry identity.
- `LvcTrackingOverlay`: installed placement/verifier handle.

## Responsibilities

- Build semantic overlay schematics.
- Write generated cache files under `.git/lvc-cache/`.
- Reuse revision-aware cache files through Litematica.
- Add/remove/focus `SchematicPlacement` instances.
- Preserve current Litematica placement origin in memory during reload.
- Seed first-load origin from project creation or current Litematica state.
- Attach schematic verifier when needed.

## Placement Rules

- No `local.json`.
- No project-repo placement persistence.
- Loaded placement list persistence belongs to Litematica.
- Project editor origin edits call into placement-origin update flow.
- Manual origin is separate versioned site metadata. Its world marker is the
  loaded placement origin plus the project-relative manual origin; moving it
  must not move or rebuild the tracking placement.
- Project Editor sub-region selection is optional single-selection Litematica
  placement state. Keep the editor list, configure hotkey, and placement
  selection synchronized, including the intentional no-selection state, and
  preserve a valid selection while replacing an overlay during a rebuild.
- Edit Project mode exposes the selected current-revision tracking-overlay
  sub-region or manual origin as a transient `AreaSelection`. Sub-regions use
  one box; the manual origin uses the explicit selection origin with no boxes.
  The selection is owned by Gitmatica, is never registered with or persisted by
  Litematica's selection manager, and must not appear in Litematica's
  area-selection browser.
- Execute Operation applies the transient target. Sub-region bounds schedule
  the normal working-tree overlay rebuild; manual-origin changes update only
  versioned metadata and its marker. Switching modes, unloading the world, or
  selecting another target discards the unapplied draft. Shared hits prefer the
  focused target, then the visible origin, then sub-regions alphabetically.
- Edit Project mode must ignore regular Litematica area selections and do
  nothing without a selected current Gitmatica tracking overlay. Parent-revision
  previews and unrelated schematic placements are not editable through it.
- Closing a project drops Gitmatica's remembered placement state.
- Reopening or reinitializing the project manager reuses an existing tracking
  overlay without changing the user's selected Litematica placement. Focus only
  from flows that explicitly need to select the project overlay.
- World mutations that leave HEAD and the project manifest unchanged, including
  direct and remote discard, reuse the existing overlay and restart its verifier.
  A full rebuild is only the fallback when the current overlay cannot be reused.
- Creating, renaming, resizing, or removing a sub-region refreshes the editor's
  live list and forces an asynchronous overlay rebuild from the dirty
  working-tree manifest so the new structure appears before a version is
  committed. Ordinary reloads continue using those dirty overlay definitions;
  unrelated tracked-file changes retain committed-HEAD overlay contents.
- Dirty structural states, including undo with keep changes, build a hybrid
  overlay: sub-region definitions come from the working tree while block
  content comes from committed HEAD. Positions introduced by expanded or new
  working-tree sub-regions therefore default to air until committed content
  covers them.
- Discard compares working and committed sub-region definitions before the Git
  reset. It may reuse the overlay for block-only changes, but must rebuild the
  committed overlay after a name or bounds change.

## Cache Rules

- Use generated overlay files and descriptor metadata.
- Generated file names use the 8-character source commit token plus a stable
  identity hash. The identity covers the source revision, site, dimension,
  placement origin, loot-preview context, Minecraft data version, and cache
  format version.
- Manual origin is deliberately excluded from the generated tracking-overlay
  identity because it does not change schematic content or placement geometry.
- Do not rely on path equality to a single fixed file.
- Keep cache and descriptor writes atomic/crash-resistant.
- Retain up to four recently used entries and 512 MiB per project. Prune only
  cache files that are not current, loaded, or protected.
- Change Viewer revision previews use the same generated cache and replacement
  lifecycle. `Current` reads the captured HEAD commit, while `Parent` reads its
  first parent. A root commit's parent preview is an air schematic built with
  the root manifest's complete sub-region layout.
- Revision-preview descriptors identify whether the loaded overlay is current
  or parent. A parent-preview cache must never pass the normal current-HEAD
  cache reuse check.

## Entity Race

The ghost entity bug was caused by stale async Litematica rebuild tasks. The
standalone fix lives in `MixinPlacementManagerTaskRebuild`, which targets
Litematica's `PlacementManagerTaskRebuild`; it does not belong in repeated
Gitmatica cleanup logs. See `11-litematica-touchpoints.md`.

## Large-Overlay Performance

Large projects (roughly 1,500 or more LVC subchunks) exposed avoidable work in
the semantic-overlay pipeline. The optimized path removes that work without
raising the existing 6 ms client-tick budget.

- Traverse `LvcChunk` masks and palettes directly. Do not build a boxed
  per-block `ChunkView` map or allocate position/`TargetBlock` objects unless a
  filtering predicate requires them.
- Split dense-chunk population into batches of at most 512 tracked blocks and
  retain a small bounded decoded-object cache keyed by immutable SHA-256 IDs.
- Use revision-aware cache entries keyed
  by the source commit, site, dimension, placement origin, world/loot-preview
  context, Minecraft data version, and an explicit overlay-cache format
  version.
- Keep at most four recently used entries and 512 MiB per project. Always
  protect the active entry, publish files atomically, and rebuild corrupt or
  incompatible entries.
- A narrow Litematica factory creates an empty schematic already bound to its
  final cache path. Register the completed in-memory schematic instead of
  writing it and immediately reading it back.
- Cache file I/O runs on one low-priority daemon worker. Semantic population,
  placement, verifier, world-sensitive work, and final publication remain on
  their required threads. Keep the old overlay visible until replacement
  succeeds.
- Cancellation, logout, a changed HEAD, or a newer task epoch discard the
  prepared result and temporary files without publishing a stale placement.
- Debug logs include timings for cache lookup, read/decode, population, cache I/O,
  placement registration, total elapsed time, cache hits, and the largest
  client-tick slice.

Integration coverage verifies block/entity content paths, root-parent air
overlays, bounded batches, deterministic placement-aware identities, atomic
replacement, and descriptor recovery. Litematica scheduling, cancellation,
relogs, eviction, and real 1,500-subchunk performance still require manual
Minecraft validation.

This work is shared by singleplayer and multiplayer client overlays. It must
not change `.lvcchunk`, repository metadata, Servux protocol behavior, overlay
origin semantics, Litematica renderer thread settings, or the stale rebuild
guard in `PlacementManagerTaskRebuild`.
