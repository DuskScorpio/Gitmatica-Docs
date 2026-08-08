# Tasks And Recovery

Long operations run as Litematica scheduler tasks.

## Base Infrastructure

- `LvcTaskBase<T>`: lifecycle, callbacks, registry release.
- `LvcChunkedTaskBase<T>`: chunked per-tick execution.
- `LvcTaskRegistry`: single active foreground operation plus background guards.
- `LvcTaskScheduling`: client/server scheduler selection.
- `LvcTaskCallbacks<T>`: success/failure/abort callbacks.
- `LvcOperationHandle`: operation id/name/repo.
- `LvcRemoteApplyRequest`: typed remote apply requests; avoid nullable mode
  parameter bags.
- `LvcTickFreezeScope`: crash-safe in-memory ownership of the singleplayer
  freeze/restore window and bounded scheduled-tick cleanup.
- `LvcCommandPasteConfigScope`: restores temporary Litematica paste settings.

Foreground tasks and background overlay loads are mutually exclusive. Treat an
overlay load as an active operation: normal project actions must wait for it and
must never abort it. World unload remains the only generic cancellation path.

World unload is a hard task-lifecycle boundary. It advances the task epoch and
clears foreground and background ownership before stopping scheduled tasks.
Callbacks from the previous epoch must not run against the next world session.
Every Gitmatica scheduler task must implement `LvcWorldTask`; task types that do
not extend `LvcTaskBase` must also epoch-guard their completion listeners.

`LvcTaskBase` retains a releasable operation lock until its callback begins on
the client thread. This makes the handoff from a completed world/repository task
to its background overlay reload atomic with respect to user input.

## Crash/Recovery

- `LvcOperationJournal`: records in-progress operations under Git-local data.
- `LvcRefreshMarker`: marks overlay refresh work that must survive until cleanup.
- `LvcInterruptedOperationPrompts`: UI prompts for recovery-sensitive failures.

Transient tracking-overlay removal writes the refresh marker before removing
the Litematica placement. The marker survives logout or task cancellation and
is cleared only after a replacement overlay is installed. Explicit project
close does not write it.

If an operation mutates world or repo state over multiple ticks, it likely needs
journal coverage.

`LvcProjectRecoveryWorkflow` owns interrupted-operation prompts, rollback, and
restart scheduling. Keep ordinary project actions in their focused workflows;
do not grow recovery back into a generic task-actions facade.

## Main Tasks

- Init: `LvcSemanticInitProjectTask`
- Save/update: `LvcSemanticCommitTask`
- Scan: `LvcSemanticScanTask`
- Checkout: `LvcSemanticCheckoutTask`
- Discard: `LvcSemanticDiscardTask`
- Clear: `LvcSemanticClearTask`
- Export: `LvcSemanticExportTask`
- Overlay load/reload: `LvcSemanticOverlayTask`
- Multiplayer apply: `LvcRemoteServerApplyTask`

Always inspect the task's `Phase` enum before changing behavior.
