# Stale Litematica Rebuild Entities After Overlay Swaps

## Summary

Gitmatica overlay swaps could leave old schematic entities rendered as ghost hitboxes after checkout, discard, save-version, or project close. The exported overlay for the new version was correct, but Litematica's schematic world could still contain entities from a previous overlay version.

The confirmed cause was stale asynchronous Litematica rebuild work committing entity state after the placement had already been removed from the live placement list.

## User-Visible Symptoms

- Switching from a project version with entities to a version without those entities left old entity outlines visible.
- Turning schematic rendering off stopped blocks from rendering, but stale entity hitboxes could remain.
- Closing the Gitmatica project did not always immediately remove the stale hitboxes.
- Flying away or relogging could make the stale hitboxes disappear because Litematica eventually refreshed or rebuilt the affected schematic chunks.

## Scope

Confirmed during multiplayer Servux testing with the `540P Barterting` project. The failure is not believed to be specific to Servux. Servux made it easy to reproduce because checkout/discard operations rapidly remove and reload Gitmatica tracking overlays while Litematica placement-manager work is still running.

The issue is about Litematica schematic-world entity state. It is not about:

- exported `.litematic` contents,
- Gitmatica semantic storage contents,
- project origin persistence,
- the tracking overlay file path by itself,
- `local.json`, which no longer exists.

## Evidence

Diagnostics added to `LvcTrackingOverlayService` showed the following sequence:

1. Before removal, the old overlay placement was loaded and `WorldSchematic` had entities.
2. Gitmatica cleanup successfully removed schematic entities:
   - `worldEntities=0`
   - `placements=0`
3. Shortly afterward, entities reappeared while the live Litematica placement list was still empty:
   - `worldEntities` increased again
   - `placements=0`
   - piglin diagnostics showed `matchingPlacements=[]`
4. Litematica placement-manager worker threads logged entity creation from NBT after Gitmatica had removed the old placement.

After adding the rebuild-task guard, logs showed:

- `PlacementManagerTaskRebuild: ignored [...] stale placements for chunk [...]`
- delayed removal diagnostics stayed at:
  - `worldEntities=0`
  - `placements=0`
  - `piglins=0`

That confirms old async rebuild tasks were the source of the stale entity state.

## Root Cause

Litematica has multiple pieces of placement state:

- the live placement list, exposed by `SchematicPlacementManager.getAllSchematicsPlacements()`;
- the chunk-to-placement index, exposed internally by `getAllSchematicsTouchingChunk(...)`;
- asynchronous placement-manager rebuild tasks that build schematic chunks and spawn entities into `WorldSchematic`.

During Gitmatica overlay swaps, an old placement can be removed from the live placement list while Litematica rebuild tasks for chunks touched by that placement are already queued or running.

Before the fix, `PlacementManagerTaskRebuild` trusted `getAllSchematicsTouchingChunk(...)` directly. A stale task could therefore:

1. see an old placement through the chunk-touching index,
2. build a proto chunk from that old placement,
3. replace the schematic chunk,
4. spawn the old placement's entities into `WorldSchematic`,
5. do this after Gitmatica had already removed the placement and cleaned entities.

The live placement list was empty, but the schematic-world entity lookup was repopulated by late async work.

## Why The Cache File Path Was Not The Root Cause

The issue reproduced even after switching Gitmatica tracking overlays to unique cache file paths. Logs showed stale entities returning when no active placement matched them. That means the stale entities were not caused by overwriting the same `.litematic` file path.

Unique cache files still provide useful crash-resistance and schematic-identity isolation, but they are not the fix for this entity bug.

## Implemented Fix

The standalone addon applies the fix through
`me.soullessjump.lvc.mixin.placement.MixinPlacementManagerTaskRebuild`, targeting
Litematica's `PlacementManagerTaskRebuild`.

Behavior added:

- Treat a chunk-touching placement as active only if it is still present in `SchematicPlacementManager.getAllSchematicsPlacements()`.
- Ignore stale chunk-touching placements that have already been removed from the live placement list.
- If no active placements touch the chunk, unload schematic entities before unloading the chunk.
- After building a proto chunk, re-check the active placement set before replacing the real schematic chunk and spawning entities.
- If the active placement set changed during the rebuild:
  - abort the stale rebuild;
  - unload entities/chunk if no placements remain;
  - otherwise schedule a fresh rebuild.

This prevents stale async work from committing old entity state after placement removal.

## Safety Analysis

The fix changes Litematica behavior globally, but only in the placement-manager rebuild path.

Expected unchanged behavior:

- Stable loaded placements still rebuild normally.
- Enabled placements still place into proto chunks and spawn entities as before.
- Rendering settings, schematic loading, placement origins, and saved placement lists are not changed.

Intentional changed behavior:

- Removed placements are ignored even if a stale chunk-touching entry is still visible to an async rebuild task.
- Chunks with no active placements have schematic entities removed before chunk unload.
- Rebuild work that races with placement removal/modification is aborted or rescheduled instead of committing stale state.

This is aligned with Litematica's intended model: schematic entities should only exist for currently active loaded placements.

## Residual Risks

- The current diagnostic logging is too verbose for final alpha builds and should be removed or reduced.
- The mixin uses narrow Litematica internals because the race exists inside
  Litematica's async rebuild worker. A delayed Gitmatica cleanup would be more
  brittle and would not prevent stale work from committing.
- The guard checks placement object membership in the live placement list. It handles removal races. It does not fully solve every possible race where a placement object remains live but its internal origin/subregion state mutates during a rebuild. That is a broader Litematica thread-safety concern.
- Repeated ignored-stale-placement logs show that Litematica can retain stale chunk-touching entries long enough for multiple worker tasks to observe them. The guard prevents stale commits, but the chunk-touching index cleanup behavior may still deserve upstream review.

## Recommended Finalization

For Gitmatica alpha:

1. Keep the Litematica rebuild-task guard.
2. Keep the explicit entity unload before chunk unload in the no-active-placement branch.
3. Remove or downgrade the temporary Gitmatica piglin diagnostics.
4. Remove the temporary stale-placement debug log or gate it behind an existing debug mode only.
5. Keep unique tracking overlay cache files for now. They were not the root cause, but they avoid file-identity ambiguity and are already crash-hardened.
6. Do not revert to same-file overlay rewriting as part of this bug fix. Treat overlay cache architecture as a separate follow-up decision.

Longer term:

- Consider upstreaming the Litematica rebuild-task guard as a general race fix.
- Consider reducing the local Litematica patch to the smallest upstreamable form after diagnostics are removed.
- If Gitmatica eventually moves to in-memory overlays, keep the same active-placement guard because the stale async rebuild race is independent of disk cache layout.

## Regression Test Plan

Manual test:

1. Open a Gitmatica project with entities in the current overlay.
2. Checkout or discard to a version without those entities.
3. Turn schematic rendering off and on.
4. Close the project.
5. Confirm old entity hitboxes do not remain.
6. Confirm logs do not show `WorldSchematic` repopulating entities while `placements=0`.

Additional cases:

- Repeat in singleplayer.
- Repeat in multiplayer with Servux.
- Repeat while rapidly switching between two commits.
- Repeat with a normal non-Gitmatica Litematica schematic to confirm ordinary load/unload behavior remains unchanged.
