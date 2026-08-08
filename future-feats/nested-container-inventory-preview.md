# Nested Container Inventory Preview

## Problem

Component-aware comparison can identify that a shulker-box item changed, but it
does not explain which slots inside that item are different.

## Planned Behavior

- Let the user open a nested Before/After preview from a container item whose
  `DataComponents.CONTAINER` payload differs.
- Reuse MaLiLib inventory backgrounds, slot layout, stack rendering, and tooltip
  behavior.
- Apply the existing added, removed, changed, and unchanged slot colors inside
  the nested preview.
- Bound preview depth and screen usage so deeply nested containers cannot create
  unbounded UI or unreadable layouts.
- Preserve the outer inventory selection while a nested preview is inspected.

This is a presentation feature. Component-aware inventory comparison remains
the source of truth for deciding whether the outer slot changed.

## Verification

- Cover added, removed, and changed nested stacks.
- Cover empty and partially filled shulker boxes.
- Confirm hover tooltips and preview placement at small GUI scales.
