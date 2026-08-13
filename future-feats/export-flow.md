# Version Export Flow

## Implementation Status

Implemented. Project Manager history entries and loaded Gitmatica overlays open
the exact-revision Litematica save browser described below.

## Screen

Every Gitmatica export action opens a screen extending Litematica's
`GuiSchematicSaveBase`. Reuse its file browser, remembered save directory,
filename validation, **Save Schematic** button, Shift-to-overwrite behavior,
and parent-screen navigation without modifying Litematica.

- Title it **Export Gitmatica Version: `<project> (<short-id>)`**.
- Prefill the editable filename with `project-shortcommit`.
- Share Litematica's `schematic_save` browser context.
- Hide the world-capture checkboxes, which do not apply to an exact commit.
- Add **Export with Manual Origin** on the right, unchecked each time.

Saving sanitizes the filename, adds `.litematic`, rejects invalid destinations,
and refuses to replace an existing file unless Shift is held. On success,
refresh the browser and keep the normal Litematica save-screen behavior.

## Revision Selection

Capture an immutable export target when opening the screen: repository, exact
commit, site, project name, and suggested filename.

- Project Manager exports its selected commit and active/default site.
- Loaded Gitmatica rows use the overlay descriptor's commit and site, never the
  cached hybrid schematic.
- Current dirty overlays therefore export HEAD; parent and historical overlays
  export their represented commit.

## Export Task

On **Save Schematic**, capture the chosen directory, filename, Shift overwrite
state, and manual-origin checkbox. Acquire the operation guard at this point,
then reconstruct and write the commit asynchronously.

Keep the browser responsive, prevent duplicate submission, and refresh it if
still open when the task finishes. Put orchestration in a focused export
workflow instead of growing the project controller.

## Origin Policy

Use an export-specific schematic-origin policy that does not affect tracking
overlay construction:

- **Unchecked:** omit the explicit `AreaSelection` origin so Litematica uses
  the component-wise minimum of the revision's subregion bounds.
- **Checked:** use that commit and site's stored project-relative
  `manual_origin`; v1 sites use implicit `[0, 0, 0]`.

The checkbox is independent of in-game origin visibility. Origin choice must
not alter committed blocks, entities, repository state, or overlay identity.

## Verification

- Cover automatic and manual offsets, including v1, negative, and
  outside-bounds origins.
- Cover selected, parent, historical, and dirty-HEAD revision resolution.
- Cover editable names, nested directories, extensions, invalid input, and
  Shift-only overwrite.
- Verify capture controls are absent and the manual-origin option defaults off.
- Verify asynchronous completion, duplicate-submit prevention, browser refresh,
  exact blocks/entities, and a clean repository.
