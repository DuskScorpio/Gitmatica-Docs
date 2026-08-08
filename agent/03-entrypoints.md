# Entrypoints

## Subsystem Entrypoints

- `LvcProjectCatalog`: project discovery, summaries, deletion, and cache.
- `LvcSemanticProjectOperations`: semantic create/commit/scan/export/update.
- Focused `me.soullessjump.lvc.git.*` classes: branch/history/remote/merge/undo work.
- `LvcTrackingOverlayService`: tracking-overlay orchestration.

Call the canonical owner directly. Do not add a second result namespace or a
cross-package project/Git facade.

## UI Entrypoints

- Main project screen: `GuiLvcProjectManager`
- Project browser: `GuiLvcProjectBrowser`
- Project editor: `GuiLvcProjectEditor`
- Workflow coordinator: `GuiLvcProjectController`
- Button command wiring: `GuiLvcProjectCommandListeners`
- Interrupted operations: `LvcProjectRecoveryWorkflow`
- Action orchestration: focused `Lvc*Workflow` classes

## Task Entrypoints

- Scheduling helpers: `LvcTaskScheduling`
- Active-operation guard: `LvcTaskRegistry`
- Base classes: `LvcTaskBase`, `LvcChunkedTaskBase`
- Callback wrapper: `LvcTaskCallbacks`

## Litematica Entrypoints

Use the matching classes under `codebases/litematica` as reference source.
Standalone integration is owned by `me.soullessjump.lvc.mixin` and
`me.soullessjump.lvc.integration.litematica`.

- Main menu integration: `GuiMainMenu`
- Loaded schematics: `GuiSchematicLoadedList`
- Placements: `GuiSchematicPlacementsList`
- Save/load/browser patterns: `GuiSchematicBrowserBase`, `GuiSchematicSaveBase`
- Placement management: `SchematicPlacementManager`
