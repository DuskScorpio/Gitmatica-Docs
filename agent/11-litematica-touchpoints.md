# Litematica Touchpoints

Litematica is an external runtime dependency of the standalone addon. Inspect
`codebases/litematica` for reference, but implement addon behavior through
Gitmatica's narrow mixins, access interfaces, and integration classes unless the
user explicitly requests an upstream Litematica change.

## Placement Manager

- Reference owners: `SchematicPlacementManager`, `SchematicPlacement`,
  `SubRegionPlacement`, `PlacementManagerDaemonHandler`,
  `PlacementManagerTaskRebuild`, and `PlacementManagerTaskFixer`.
- Addon touchpoints:
  `me.soullessjump.lvc.mixin.placement.MixinPlacementManagerTaskRebuild`,
  `MixinPlacementManagerTask`, and
  `integration.litematica.placement.GitmaticaPlacementDaemon`.

## Schematic World

- Reference owners: `WorldSchematic`, `ChunkManagerSchematic`,
  `ChunkSchematic`, `ProtoChunkSchematic`, and `SchematicEntityLookup`.
- Addon render/lifecycle hooks live under `me.soullessjump.lvc.mixin.render` and
  `me.soullessjump.lvc.mixin.lifecycle`.

## Edit Project Tool Mode

- `MixinToolMode` appends Gitmatica's runtime-only `EDIT_PROJECT` enum value;
  use `edit_project` as its serialized name and retain Litematica's normal
  cycling behavior.
- `MixinToolHud` gives Edit Project the normal placement summary followed by
  singular Corner Mode and Mode lines. Modified flags include applied placement
  and HEAD-to-working structural changes, never unapplied transient box edits.
- `MixinSelectionManager` supplies the transient Gitmatica `AreaSelection` only
  while that mode is active, hides regular selection IDs, and blocks changing
  the regular selection. Never insert the transient selection into Litematica's
  stored-selection maps.
- `MixinDataManager` clears the edit session on tool-mode transitions, while
  `MixinKeyCallbackHotkeys` resolves clicks on other tracking-overlay corners
  before allowing Litematica's normal corner-edit behavior.
- Corner-hit candidates are sorted by sub-region name. Equal-distance shared
  corners intentionally retain the alphabetically first candidate.

## Known Race Fix

`MixinPlacementManagerTaskRebuild` must filter chunk-touching placements
through the live placement list. A stale task must not rebuild or spawn
entities for a removed placement.

Also keep entity unload before chunk unload when no active placements remain.

## Rule For Future Changes

If a Gitmatica bug appears to require deeper Litematica access:

1. Prove the bad state exists in Litematica-owned structures.
2. Prefer a public API, then a narrow access interface or injection.
3. Keep behavior general enough to upstream, without patching the local
   reference checkout.
4. Add or run regression coverage where practical.
