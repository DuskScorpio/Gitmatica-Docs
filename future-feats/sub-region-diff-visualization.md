# Subregion Diff Visualization

## Comparison

Compare the working manifest with HEAD. Repeated applies keep HEAD as the
baseline and the latest applied bounds as the working state. This feature does
not add historical Before/After comparison.

Use subregion names as identity:

- Changed: HEAD bounds orange; working bounds cyan.
- Added: working bounds green.
- Removed: HEAD bounds red.
- Renamed: removed under the old name and added under the new name.

Show all applied structural bounds only in Tool Mode 10 until commit or discard.
Block overlays and Change Viewer entries remain visible normally in other tool
modes. Do not draw movement arrows.

An unapplied edit keeps the applied bounds visible and uses Litematica's normal
area-selection styling for its draft. Recompute structural state only on apply.
Applied bounds follow Litematica's Placement Boxes toggle; the draft follows its
Area Selection Rendering toggle. Status colors take precedence over selection
colors after apply.

## Coverage Diffs

Compare the union of all HEAD bounds with the union of all working bounds. A
position covered by another subregion never becomes added or removed.

- **Retired coverage:** Render each non-air block tracked in HEAD as a red ghost
  and a Removed block diff, regardless of the current world block. Ignore later
  world edits there and never clear the physical blocks.
- **New coverage:** Compare against air. Each current non-air block is a green
  Added diff; changing it updates the Added state, and returning it to air
  removes the diff.

HEAD air and blocks newly placed in retired coverage produce no diff. An exact
delete-and-add replacement shows red and green bounds but no block diffs because
overall coverage is unchanged.

Retired containers produce only Removed block entries, not inventory entries.
Entity diffs are out of scope.

## Change Viewer

Feed structural block diffs through the existing six-neighbor groups, filters,
selection highlights, Hide, and Reset Hidden behavior. Do not add structural
subregion rows or categories.

## Deferred: Coincident Outlines

Render each subregion outline normally for now. Multiple dirty subregions with
identical or overlapping bounds may obscure or z-fight with each other. Do not
add offsets, aggregation, striping, or special precedence in this version;
revisit a scalable presentation separately.
