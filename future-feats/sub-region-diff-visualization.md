# Sub-region Diff Visualization

## Scope

Show structural changes to sub-region bounds without redefining existing Change
Viewer block diff behavior.

## Comparison Base

- Working-tree edit view compares current edits against the latest commit.
- Diff viewer compares After against Before.

## Bounds Visualization

- Changed sub-region:
  - show previous bounds in orange;
  - show new bounds in blue;
  - show an orange arrow from previous bounds to new bounds.
- Added sub-region:
  - show only the new bounds in green.
- Deleted sub-region:
  - show only the previous bounds in red.

## Block Diffs

Use existing Change Viewer block added/removed/changed behavior where possible.

Sub-region edits can change which positions are tracked. When a previously
tracked position becomes untracked, it should appear as removed from tracked
schematic content even if the physical world still contains a block there.

This covers shrinking or deleting a sub-region while the world blocks remain in
place.
