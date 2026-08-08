# GUI And MaLiLib

Gitmatica UI should look like Litematica UI.

## Mandatory Rule

Before adding or changing Gitmatica UI, inspect nearby Litematica GUI code that
uses MaLiLib. Match its patterns first.

The local reference checkout is under
`codebases/litematica/src/main/java/fi/dy/masa/litematica/gui/`.

## Litematica UI References

- Main menu: `GuiMainMenu`
- Loaded schematics: `GuiSchematicLoadedList`
- Placements: `GuiSchematicPlacementsList`
- Save dialogs: `GuiSchematicSaveBase`, `GuiSchematicSaveExported`
- Browser base: `GuiSchematicBrowserBase`
- Verifier: `GuiSchematicVerifier`
- Widgets: `fi.dy.masa.litematica.gui.widgets.*`

## Gitmatica UI Owners

- `GuiLvcProjectManager`: main project screen.
- `GuiLvcProjectController`: state and actions.
- `LvcProjectRecoveryWorkflow`: interrupted-operation recovery only.
- `LvcVersionDeleteWorkflow`: latest-version delete/undo flow.
- `GuiLvcProjectCommandListeners`: button listeners.
- `GuiLvcProjectEditor`: regions/origin editor.
- `GuiLvcChangeViewer`: live local/unsaved change inspection.
- `WidgetLvcCommitHistoryList`, `WidgetLvcCommitMetadataPanel`: project screen
  history and metadata panels.
- `WidgetLvcProjectRepositoryBrowser`: the repository/file browser panel inside
  the `GuiLvcProjectBrowser` page.
- `WidgetListLvcChangeViewerEntries`, `WidgetLvcChangeViewerEntry`: Change
  Viewer list and row rendering.
- `Lvc*Workflow`: action-specific UI flow orchestration.
- `me.soullessjump.lvc.gui.widgets`: custom dropdown/list widgets.

## Reusable Gitmatica UI Pieces

- `GuiLvcPanelDialog`: centered outlined MaLiLib dialog shell with parent
  rendering and correct widget/button layering.
- `LvcButtonRowLayout`: exact dynamic button sizing and spacing.
- `LvcGuiText`: wrapping and ellipsis behavior.
- `WidgetLvcVerticalScrollbar`: shared scrollbar rendering, hit testing, and
  dragging.
- `WidgetLvcBlockPosEditor`: reusable X/Y/Z editor with modifier-aware nudge
  controls and an optional move action.
- `WidgetLvcSearchableListDropdown` and `WidgetLvcPopupHost`: searchable
  dropdown content and shared modal popup routing.
- `WidgetLvcBranchControls`: Project Manager Push/Pull/branch-control strip.
- `LvcChangeViewerStateStore`: weak verifier-scoped Change Viewer state.
- `LvcChangeEntry`: Change Viewer presentation model.
- `WidgetLvcBlockStateDiffTooltip`: block mismatch tooltip presentation.

## Patterns

- Use `StringUtils.translate` and existing translation key style.
- Use `ButtonGeneric`, `GuiListBase`, `WidgetListBase`, `WidgetListEntryBase`
  before creating new widget concepts.
- Name whole pages `Gui*` and embedded visual/input components `Widget*`; use a
  panel-specific name when both represent the same feature.
- Keep button/list sizing consistent with neighboring Litematica screens.
- Keep controller and semantic-service logic out of widgets. Pass narrow
  callbacks or a small view contract into reusable components.
- Put multi-step actions in focused workflows instead of adding more methods to
  `GuiLvcProjectController`.
- Do not add explanatory in-app text when state and labels can carry the UI.
