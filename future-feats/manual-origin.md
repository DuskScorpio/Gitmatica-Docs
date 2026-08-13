# Manual Origin

## Implementation Status

Manual-origin storage, Project Editor controls, Edit Project interaction,
working/HEAD markers, and merge behavior are implemented. Pending-draft
mutation blocking remains deferred. The export behavior described below is
implemented.

## Model

Each site versions one `manual_origin: [x, y, z]` relative to Gitmatica's fixed
project origin. It is always a coordinate, never an enabled/disabled mode, and
does not affect the tracking placement, overlay, verification, or Change
Viewer.

```text
world marker = placement origin + manual_origin
exported region position = project-relative position - manual_origin
```

Moving the local placement moves the marker without dirtying the repository.

## Storage

The migration framework introduces `lvc-manifest-v2` and stores
`manual_origin` per site.

- New projects use v2 with `[0, 0, 0]`, regardless of the source selection's
  origin mode.
- V1 sites read as `[0, 0, 0]` without rewriting history.
- Applying a changed origin upgrades a v1 working manifest. Once introduced,
  v2 and the field remain even if the origin returns to zero.
- Older Gitmatica versions reject v2 rather than dropping its data.
- Origin edits participate in dirty, commit, discard, checkout, undo, history,
  and merge behavior, but never create Change Viewer entries.

Exclude `manual_origin` from tracking-overlay content and identity. Historical
overlays remain anchored to their existing fixed project origin.

## Project Editor

Keep **Placement Origin** unchanged. Add a world-coordinate **Manual Origin**
XYZ editor with normal nudge controls and **Move to Player**. Reflow both origin
editors on narrow screens.

Add **Show Manual Origin**, remembered client-locally per project and never
written to the project repository. It only controls in-world visibility and
does not change focus, drafts, or export behavior.

- The visibility toggle works in every tool mode.
- Origin controls remain visible but read-only while the marker is hidden,
  Tool Mode 10 is inactive, or Execute Operation is unbound.
- Selecting an editable origin control focuses the origin and discards any
  pending subregion draft.
- Hiding the marker preserves an existing origin draft.
- Execute Operation inside Project Editor does nothing.

## In-Game Editing

When shown, render every enabled loaded project's working origin in
Litematica's normal orange in every tool mode. Outside Tool Mode 10 it is
view-only.

Tool Mode 10 (**Edit Project**) exposes either one subregion or the origin
through Gitmatica's transient Litematica selection:

- Selecting the origin discards the subregion draft; selecting a subregion
  discards the origin draft.
- Use Litematica's normal middle-click selection, tool movement, nudging,
  selected cyan, and depth testing.
- Prefer the focused target at shared hits, then the visible origin, then other
  subregion corners alphabetically.
- Block add/delete-box, set/delete-origin, and whole-selection translation.
  Origins and subregions must move independently.

Use Litematica's configured Execute Operation hotkey as the only apply action
and respect `executeRequireTool`. If unbound, editing is read-only.

Only a genuinely changed draft blocks local repository mutations. Read-only
history, inspection, verification, and export remain available. Leaving Tool
Mode 10, changing overlays, or unloading the world cancels the draft.

Match Litematica's signed `BlockPos` coordinate freedom, including negative and
out-of-world positions, but reject arithmetic overflow before apply or export.

## Applied Move Indicator

After apply, compare HEAD's coordinate with the working coordinate:

- Keep the working origin orange and render the old HEAD origin as a
  depth-tested yellow outline.
- Show both in every tool mode whenever **Show Manual Origin** is enabled.
- During a pending origin move, show only the selected cyan draft.
- Render no arrow and no old marker when the coordinates match.
- Suppress the Gitmatica placement-origin marker when it overlaps the manual
  origin marker.

Derive markers from HEAD and the working manifest so they survive relogging and
disappear after commit or discard without opening Project Manager. Render all
projects whose local visibility setting is enabled.

## Export

This flow is implemented.

Every Gitmatica export opens a popup with:

- An editable filename prefilled as `project-shortcommit.litematic`.
- **Export with Manual Origin**, unchecked each time.
- Overwrite confirmation when the destination exists.

Unchecked exports omit the explicit selection origin, making Litematica use the
component-wise minimum of the exported revision's subregion bounds. Checked
exports use that revision's `manual_origin`; v1 revisions use `[0, 0, 0]`.

Always rebuild the exact represented commit. Project Manager exports its
selected commit, parent and historical rows export their displayed commit, and
a dirty current overlay exports HEAD while ignoring working changes.

## Merge

Merge each site's complete XYZ coordinate atomically. Missing v1 and v2 zero
are semantically equal. Keep equal values or a one-sided change; conflict only
when both sides move the same site's origin differently. Resolve conflicts with
the existing global Base/Yours/Incoming choice. Format upgrades are not merge
conflicts, and v2 remains sticky once introduced.

## Verification

- Cover v1/v2 loading, sticky upgrades, round trips, and atomic merges.
- Cover automatic/manual exact-revision exports, negative and outside-bounds
  origins, filenames, and overwrite confirmation.
- Cover exclusive focus, hotkey gating, GUI no-op behavior, draft blocking,
  marker visibility, relog restoration, multiple projects, and placement moves.
- Confirm origin edits never change overlay identity, placement geometry,
  verification, or Change Viewer output.
