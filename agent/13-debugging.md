# Debugging

## Logs

Common client logs:

- `run/logs/latest.log`
- `run/logs/debug.log`

Servux test server logs may live under:

- `run/servers/*/logs/latest.log`

## Gitmatica Diagnostics

Use `LvcDiagnostics` for operation-scoped logs. Prefer debug-level logs unless
the message is important to normal users.

User-facing debug logging is controlled by Gitmatica Configs -> Generic ->
`debugLogging`. The setting persists in `config/gitmatica.json`. Like
Litematica's `debugLogging`, enabled `LvcDiagnostics.debug(...)` messages are
written at INFO level with a `[DEBUG]` prefix so they appear in `latest.log`.
Development builds also enable diagnostics through Litematica's
`Reference.DEBUG_MODE`.

`LvcConfigs` propagates the effective setting into primitive runtime state for
`LvcDiagnostics`. Do not make low-level diagnostics initialize MaLiLib config
objects directly: semantic repository integration tests run before Fabric's
game directory exists.

Temporary investigation logs are allowed, but remove or gate them before
finalizing a fix.

## Useful Triage Questions

- Is the bad state in Gitmatica semantic storage?
- Is the exported `.litematic` correct?
- Is Litematica's live placement list correct?
- Is `WorldSchematic` stale?
- Is a Litematica async task still running?
- Does the issue reproduce after a full client restart?

## Entity/Overlay Bugs

For schematic entity rendering bugs, inspect:

- `WorldSchematic`
- `SchematicEntityLookup`
- `SchematicPlacementManager`
- `PlacementManagerTaskRebuild`
- `PlacementManagerTaskFixer`
- `LvcTrackingOverlayService`

Avoid leaving entity-specific dump logs in final code.
