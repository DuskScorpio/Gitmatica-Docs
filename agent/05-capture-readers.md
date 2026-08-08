# Capture Readers

## Flow

Capture is split so tasks do not care where block/entity data comes from:

1. Build a `LvcSiteWorkPlan`.
2. Create a `LvcCaptureSession`.
3. Read chunks through a `LvcWorldReader`.
4. Feed data through `LvcCaptureEngine`.
5. Publish through semantic repository or compare for scans.

## Files

- `LvcCapturePlanner`: region/chunk planning.
- `LvcSiteWorkPlan`: planned site work.
- `LvcCaptureSession`: cursor/session state.
- `LvcCaptureEngine`: captures planned chunks into semantic objects.
- `LvcWorldReader`: source interface.
- `LvcMinecraftWorldReader`: direct Minecraft world reader.
- `LvcServuxWorldReader`: Servux-backed multiplayer reader.
- `LvcBlockStateOnlyWorldReader`: sparse/block-only reader.
- `LvcServuxBulkEntityCache`: entity data batching/cache.

## Backend Choice

`LvcWorldBackend` and `LvcWorldAccess` decide whether an operation can use an
integrated-server `ServerLevel`, client-level data, or Servux data.

Any change to capture behavior must be checked for both singleplayer and
multiplayer/Servux paths.

