# Restore And Apply

Restore/apply paths differ by world backend.

## Singleplayer

Singleplayer can use integrated-server `ServerLevel` paths. Check:

- `LvcWorldAccess`
- `LvcWorldBackend`
- `LvcSemanticRestoreEngine`
- `LvcAuthoritativeClientSyncTask`
- `LvcAuthoritativeScanSync`

## Multiplayer With Servux

Multiplayer restore uses command/sparse paste paths:

- `LvcRemoteServerApplyTask`
- `LvcRemoteSparseTargetPlanner`
- `LvcSparseCommandPasteTask`
- `LvcServuxBulkRequestPlanner`
- `LvcServuxWorldReader`

## Checkout/Discard/Clear

- Checkout preflight lives under `LvcSemanticCheckoutTask.Preflight`.
- Checkout apply lives under `LvcSemanticCheckoutTask.Apply`.
- Discard uses `LvcSemanticDiscardTask`.
- Clear uses `LvcSemanticClearTask`.
- Checkout, delete-last-save, and merge clear source-only subregion coverage to
  air. Compute this as the union of source bounds minus the union of target
  bounds through `LvcRetiredCoveragePlan`; never clear old region boxes
  independently because a target region may still overlap them.
- Singleplayer checkout/discard uses one initial scan to plan direct rewrites;
  it does not run a post-operation verification scan or show a mismatch popup.
  Initial block-entity comparisons use
  `LvcChunkCodec.encodeTrackedBlockEntityContent`, not full canonical BE NBT.
- Singleplayer discard freezes the integrated server's global tick manager for
  the scan/restore lifecycle. Immediately before its first real world mutation,
  it clears scheduled block ticks, fluid ticks, and queued block events inside
  the exact tracked sub-region bounds. No-op discard scans do not clear queues,
  and task shutdown must restore the previous freeze/step state.
- Servux sparse planning uses the same tracked BE projection. Full canonical BE
  bytes are still used for paste/export payloads, but sparse dirty decisions
  must stay verifier-visible.
- Servux sparse restore must request fresh bulk data for every tracked server
  column before building its target schematic. Use a fresh-only
  `LvcServuxWorldReader`; never fall back to Litematica's reconnect-era BE
  cache because an unavailable inventory is indistinguishable from an empty
  committed inventory.

## Entity Cleanup

- Multiplayer command cleanup uses `/tp` and `/kill`. Killing item frames can
  create item-entity drops, so cleanup must also sweep `type=item` in the
  original region and holding area before paste.

When changing restore behavior, test both singleplayer and multiplayer/Servux
because entity/block availability and command execution differ.
