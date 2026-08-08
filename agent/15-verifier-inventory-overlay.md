# Verifier Inventory Overlay

This is the overlay/change UI shown inside an opened container screen, plus the
inventory preview tooltip used by the Change Viewer.

## Owner

- `fi.dy.masa.litematica.schematic.verifier.inventory.VerifierInventoryOverlay`
  owns binding, preview creation, slot coloring, ghost expected stacks, and
  tooltip preview rendering.
- `fi.dy.masa.litematica.schematic.verifier.inventory.VerifierInventoryLookup`
  finds schematic-side inventory data for a world position.
- `VerifierInventoryPreview`, `VerifierInventorySide`, and
  `VerifierInventorySlotDiff` carry the expected/found inventory comparison.
- `GuiLvcChangeViewer` renders Gitmatica's live unsaved-change list.
- `LvcSpatialDiffGroups` is the generic six-neighbor grouping engine. It accepts
  positioned changes with arbitrary payloads so commit diffs and merge conflicts
  can reuse it.
- `LvcVerifierDiffGroups` adapts live `SchematicVerifier` mismatches into the
  generic grouping engine and preserves verifier row aggregation/previews.
- `SchematicVerifier` exposes read-only snapshots of its block and inventory
  mismatch maps so spatial consumers do not depend on private mutable state.

## Open Container Flow

1. `MixinClientPlayerInteractionManager_easyPlace` records the clicked block
   position after `MultiPlayerGameMode.useItemOn`.
2. `MixinGui` observes `Gui.setScreen` and calls
   `VerifierInventoryOverlay.onScreenChanged`.
3. `VerifierInventoryOverlay.bindOpenContainer` binds the screen's non-player
   container slots to schematic inventory data at the clicked position.
4. `MixinAbstractContainerScreen` intercepts `AbstractContainerScreen`
   extraction/rendering and calls `VerifierInventoryOverlay.drawScreenStack`.
5. `MixinClientPacketListener` refreshes the bound preview when container
   content or slot packets arrive.

## Rendering

For the real opened container screen:

- `MixinAbstractContainerScreen` routes slot stacks through
  `VerifierInventoryOverlay.drawScreenStack`.
- `MixinTechUtilsInventoryOverlay` makes Tech Utils yield while Gitmatica has a
  bound screen preview. Without this ownership boundary, both mods draw
  translucent colors on the same slot: green becomes washed out, yellow mixes
  with Tech Utils red into orange, and red misleadingly appears correct.
- `VerifierInventoryOverlay.drawStackInternal` colors mismatched slots and can
  render expected missing stacks as transparent ghost stacks.
- `MixinGuiGraphicsExtractorVerifierInventory` and
  `MixinGuiRendererVerifierInventory` apply transparent item rendering state.

For preview tooltips:

- `GuiLvcChangeViewer` calls `VerifierInventoryOverlay.renderPreviewTooltip`.
- `VerifierInventoryOverlay.renderSide` uses
  `fi.dy.masa.malilib.render.InventoryOverlay.renderInventoryBackground` and
  `renderInventoryStacks`.
- Change Viewer inventory previews use a fixed, centered-above-crosshair
  position. Hover shows a preview without selecting it; left-click selection
  locks the preview so its item slots can be hovered. Locked inventory rows are
  ordered by selection time, matching Tech Utils: the most recently selected
  row is shown and deselecting it falls back to the previous selected row.
- `MixinInventoryOverlayVerifierInventory` hooks MaLiLib inventory rendering so
  verifier previews can reuse the same slot diff coloring and ghost stacks.
- The held-`I` two-column block comparison keeps Litematica's normal
  `Expected`/`Found` headings, but uses `Before`/`After` when the selected
  placement is a Gitmatica tracking-cache placement.

## Change Viewer Grouping

- Build connectivity from the union of block and inventory mismatch positions.
- Connectivity is face adjacency only: north, south, east, west, up, and down.
- Groups sort deterministically by their lowest `(x, y, z)` position.
- With the All filter, every expanded group shows these subgroups in order:
  inventories changed, blocks added, blocks removed, blocks changed, blockstate
  changed. Category filters hide nonmatching groups and subgroup rows while
  preserving original group numbers, stable selection identities, and expansion
  state. The active filter limits selected verifier highlights and Right Ctrl's
  render-through schematic overlay to visible subgroups without discarding saved
  selections. The normal depth-tested schematic overlay remains unfiltered, and
  returning to All restores Litematica's normal render-through behavior. Filtered
  render buffers carry the verifier filter revision so stale asynchronous chunk
  rebuilds are not drawn. Filter choice is remembered per verifier while that
  verifier remains live.
- Subgroup label colors come from `LvcTrackingOverlayService` marker colors.
- Hide delegates to verifier ignore state. Block rows hide every position with
  the same mismatch type and Before/After state pair; inventory rows hide only
  their specific position. The verifier retains hidden per-position snapshots
  so Reset Hidden can restore both categories immediately, including singleton
  grouped rows and state-pair ignores that survive a verifier rescan.
- The overlay revision button is placed after the category filters. It replaces
  the tracking placement through `LvcSemanticOverlayTask`, keeps the existing
  placement origin, starts the new verifier, and rebinds the open viewer to the
  replacement placement/verifier.

## Rules

- Keep MaLiLib inventory rendering as the source of truth for background, slot
  layout, stack rendering, and tooltip behavior.
- Keep finished-verifier live block rechecks inside the placement's enabled
  sub-region boxes and active verifier layer range. Block updates elsewhere in
  a touched chunk must not create mismatches or highlights.
- Do not duplicate inventory-grid layout logic in Gitmatica UI.
- Treat open-container binding as screen/session state, not persisted project
  state.
- Treat recorded block-click positions as one-shot pending state. A failed block
  open, such as a blocked shulker, must not bind inventory diffs to a later
  entity, player, or creative inventory screen.
- Binding must verify the opened non-player container matches the schematic
  inventory shape for the clicked block.
- Be careful with slot selection: player inventory and result containers are
  intentionally excluded from verifier binding.

## Litematic Inspection

Use the local inspector when container block states and block-entity NBT appear
to disagree:

```sh
python3 scripts/inspect-litematic.py /path/to/file.litematic
```

It prints each region's block entities alongside the block palette state at the
same local coordinate. This is especially useful for older schematics whose raw
block-entity records omit `id`; data fixing may otherwise infer an incompatible
type from ambiguous inventory NBT.

## Known Investigation: Session-Stale Container Contents

An unreproduced report showed one chest preview containing implausible items.
The same bad contents returned after verifier rescans, but disconnecting and
rejoining fixed them. No exception or relevant Gitmatica/Servux diagnostic was
present in the retained client logs, so the cause is not yet proven.

Current evidence points upstream of the Change Viewer:

- `SchematicVerifier.reset()` clears `inventoryMismatches`, and each verifier
  pass creates new `VerifierInventorySide` snapshots from the schematic and
  found block entities. A bad preview that returns after a rescan was therefore
  recreated from one of those worlds; it was not retained by the tooltip or
  row-selection state.
- In multiplayer, Servux bulk replies are loaded into the matching client-world
  container block entities by `EntityDataManager`. The verifier subsequently
  reads those client block entities.
- `LvcServuxBulkEntityCache` correlates an outstanding reply by `ChunkPos` only.
  It has no request generation or session token, so an overlapping or delayed
  reply for the same column is a plausible stale-data race. This remains a
  hypothesis, not a confirmed defect.
- Logout resets Litematica's entity data and explicitly clears the Gitmatica
  Servux request/reply cache. That lifecycle matches the observed recovery after
  a relog.

For the next reproduction, record the dimension and block position, whether the
`Before` or `After` side is wrong, the active world backend (especially Servux),
and copy `latest.log` before relogging. Also compare the preview with the actual
opened chest. If instrumentation is authorized, log a per-request generation,
reply transaction/column, and the source of each container NBT load; do not fix
this by retaining or refreshing Change Viewer tooltip state.
