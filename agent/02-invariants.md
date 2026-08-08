# Invariants

These are rules to protect unless the user explicitly changes direction.

## Placement

- No `local.json`.
- No clone-local placement persistence inside project repos.
- Loaded Gitmatica projects behave like normal loaded Litematica schematics.
- Origin is Litematica `SchematicPlacement` origin.
- Project editor coordinate edits must mutate Litematica placement origin.
- Stick movement must continue to mutate Litematica placement origin.
- Overlay regeneration must preserve the current placement origin in memory.

## Overlay Cache

- Current alpha direction is generated file-backed overlays under
  `.git/lvc-cache/`.
- Unique/generated cache files are allowed and preferred for now.
- Same-file rewriting is not forbidden, but it is not the current bug fix and
  should be considered separately.
- Cache descriptor writes should be crash-tolerant.

## Litematica Race Guard

`MixinPlacementManagerTaskRebuild` must keep its guard against rebuilding from
placements that are no longer in
`SchematicPlacementManager.getAllSchematicsPlacements()`.

That guard prevents stale async rebuild tasks from re-spawning old schematic
entities after overlay removal.

## Legacy

Do not preserve old `.gitignore`, `local.json`, or generated-file cleanup paths
unless the user asks. The alpha format can require existing repos to be updated.

## UI

Use Litematica/MaLiLib UI patterns first. Do not design a new visual language
inside Gitmatica screens.
