# Manual Origin

## Behavior

- Each site versions its own export origin.
- **Automatic:** use the component-wise minimum XYZ across the exported
  commit's subregion boxes, not its non-air content.
- **Manual:** use a position relative to Gitmatica's fixed project origin, not
  the bounds minimum. It may be outside all subregion bounds.
- This only affects Gitmatica `.litematic` export. It does not replace or modify
  the tracking placement's existing world origin, overlay positioning,
  verification, Change Viewer, or normal Litematica schematics.

## Coordinates

The editor uses world coordinates, but only the project-relative value is
stored:

```text
world  = placement origin + stored origin
stored = world - placement origin
exported subregion position = project-relative position - stored origin
```

Moving the local placement updates the displayed value without dirtying the
repository. The `.litematic` stores the resulting relative subregion offsets,
never a real world position.

## Storage And Migration

This feature depends on the repository migration framework and introduces
`lvc-manifest-v2`. Each site optionally stores `manual_origin: [x, y, z]`.
Presence means Manual; absence means Automatic. There is no separate boolean or
dormant position.

- New projects use v2.
- Newer Gitmatica versions read v1 and v2; migration never rewrites history.
- V1 sites load as Automatic without being rewritten; historical v1 commits
  remain exportable.
- First enabling Manual Origin in a v1 project upgrades its working manifest.
- Disabling it before any v2 commit restores v1 when no other v2-only data
  requires v2, while preserving unrelated changes.
- Committed v2 projects are never automatically downgraded.
- Older Gitmatica versions must reject v2 instead of dropping its data.

Edits write the working manifest immediately and participate in normal dirty,
commit, discard, checkout, undo, history, and merge behavior. They do not add a
Change Viewer entry.

## Project Creation

- A source Litematica selection with an explicit origin creates Manual mode at
  project-relative `(0, 0, 0)`.
- A source selection without one creates Automatic mode.

## Project Editor

Enable the existing Manual Origin toggle. The editor requires a loaded tracking
placement.

- Keep Placement Origin unchanged.
- Hide manual controls while off.
- While on, show a second inline world-XYZ editor labeled **Export Origin** with
  normal nudge controls, **Move to Player**, and **Reset**.
- Reflow both origin editors on narrow screens.
- Enabling copies the current working bounds minimum; later bounds changes do
  not move the manual origin.
- Disabling deletes the position; re-enabling calculates a fresh bounds minimum.
- Move to Player uses the player's current block position.

## Export

Build every export entirely from one committed revision, including its regions,
blocks, entities, and origin:

- Project Manager exports the selected commit.
- A loaded schematic row exports the committed revision it represents.
- Parent and historical previews export their displayed revision.
- A current or dirty hybrid overlay exports HEAD and silently ignores working
  changes.

Name the file with the existing project-name plus short-commit-ID convention,
using the exact exported commit. Do not add a dirty-state prompt.

## Merge

Treat each site's state—Automatic or Manual at one complete XYZ position—as one
atomic value:

- Keep equal states or the only side changed from the base.
- Merge independent site, subregion, and block changes normally.
- Conflict when both sides change one site's state differently, including
  Automatic versus moved, different points, or edits to different axes.

Use the existing global Base/Yours/Incoming popup. Its selected side supplies
the complete origin state; do not add an origin-specific dialog. Automatic
stores no position.
