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
- First applying Manual Origin in a v1 project upgrades its working manifest.
- Disabling it before any v2 commit restores v1 when no other v2-only data
  requires v2, while preserving unrelated changes.
- Committed v2 projects are never automatically downgraded.
- Older Gitmatica versions must reject v2 instead of dropping its data.

Applied edits participate in normal dirty, commit, discard, checkout, undo,
history, and merge behavior. They do not add a Change Viewer entry.

## Project Creation

- A source Litematica selection with an explicit origin creates Manual mode at
  project-relative `(0, 0, 0)`.
- A source selection without one creates Automatic mode.
- Empty-site creation is out of scope while that creation path is disabled.

## Project Editor

- Keep Placement Origin unchanged.
- Keep the Manual Origin toggle and a second inline world-XYZ editor labeled
  **Export Origin**, with normal nudge controls and **Move to Player**. Do not
  provide Reset.
- Reflow both origin editors on narrow screens.
- Disable all Manual Origin controls unless Tool Mode 10 has an active current
  Gitmatica overlay and Litematica's Execute Operation hotkey is bound.
- While enabled, the controls edit the Tool Mode 10 draft instead of the working
  manifest. Show status directing the user to the configured Execute Operation
  hotkey.
- Hide XYZ and Move to Player while the draft mode is Automatic.
- Enabling Manual mode copies the current working bounds minimum. Disabling it
  removes the draft position. Move to Player uses the player's block position.

## In-Game Editing

Tool Mode 10 (**Edit Subregions**) exposes the current subregion and manual
origin through one Gitmatica-owned transient Litematica `AreaSelection`:

- Include the selected subregion box when one is focused.
- Include the explicit origin only while the site or draft is in Manual mode.
- Allow an origin-only selection when no subregion is focused.
- Use Litematica's normal middle-click selection, tool-click movement, nudging,
  selected cyan, and unselected orange origin rendering.
- At a shared hit position, prefer the focused subregion's corner, then the
  origin, then other shared corners in alphabetical order.

Switching subregions discards the previous bounds draft and recreates that box
from the working manifest, but preserves the site-level origin draft. Leaving
Tool Mode 10, selecting another overlay, or unloading the world discards all
remaining drafts.

### Apply

Use Litematica's configured Execute Operation hotkey as Tool Mode 10's only
apply action; remove the hard-coded Enter action and respect Litematica's
`executeRequireTool` setting. If the hotkey is unbound, disable editing and show
binding guidance.

One press atomically validates and applies the current subregion bounds plus the
origin mode and position. Apply neither on failure and retain the drafts for
correction. Use concise actionbar messages for success, failure, and no pending
changes.

Block Save Version, checkout, discard, merge, and other repository-changing
operations while a draft is pending, with guidance to apply it or leave Tool
Mode 10 to cancel it.

## Applied Move Indicator

After a successful origin move, compare HEAD's effective origin with the
applied working origin. HEAD's effective origin is its manual position, or its
committed bounds minimum when Automatic.

- Keep the current origin in Litematica's normal unselected orange.
- Render the old origin as a darker-pink block outline and draw a matching
  directional arrow from it to the current origin.
- Depth-test the old marker and arrow like Litematica's normal origin.
- Keep HEAD as the source across repeated applies and update the working
  destination.
- Hide the indicator while another origin position is pending; show only the
  selected cyan draft until it is applied.
- Show no indicator when Manual mode is off or both positions coincide.

The indicator appears only in Tool Mode 10. It is derived from HEAD and the
working manifest, so it reappears after re-entering the mode and disappears
after commit or discard.

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
