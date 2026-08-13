# Rotated-Overlay Translation Offsets

## Goal

Allow a non-zero-rotation tracking overlay to be repositioned without
conflating its versioned transform with ordinary clone-local placement
movement. This feature depends on
[rotation-aware project changes](rotation-aware-project-changes.md).

## Coordinate Model

The unrotated placement origin is the fixed baseline. Translation exists only
for non-zero rotations and uses world axes:

```text
Original / 0 degrees:
world = baseline placement origin + canonical position

Non-zero rotation:
effective origin = baseline placement origin + rotation offset
world = effective origin + rotate(canonical position)
```

- `rotation_offset` is a versioned signed `[x, y, z]` delta.
- It is absent at Original / 0 degrees.
- Every non-zero rotation requires it, including `[0, 0, 0]`.
- Switching directly between non-zero rotations retains the offset and thus
  the effective pivot.
- Switching to Original removes it. A later switch from Original to a
  non-zero rotation starts at `[0, 0, 0]`.
- Changing the baseline placement moves the entire project locally without
  changing or dirtying the offset.

This is part of the unreleased `lvc-manifest-v2`. Missing fields in prerelease
v2 and all v1 manifests mean Original rotation with no translation. Do not add
manifest v3 or a feature-marker schema.

## Placement Separation

The active Litematica placement uses the effective origin. Gitmatica derives
the baseline without clone-local repository state:

```text
baseline origin = effective origin - rotation offset
```

- Project Editor's **Placement Origin** continues to show and edit the
  baseline.
- Normal Litematica/stick placement movement changes the effective and
  baseline origins by the same delta while preserving `rotation_offset`.
- Tool Mode 10 transform editing holds the baseline fixed and changes the
  versioned offset.
- Relogging derives the baseline from Litematica's persisted effective origin
  and the checked-out manifest offset.

## Project Editor And Tool Mode 10

Add one Project Transform section containing:

- A four-value Rotation control.
- Translation Offset XYZ fields, unavailable at Original / 0 degrees.

Rotation and offset form one exclusive transient draft. Editing the Project
Editor controls creates the draft, but Execute Operation inside the GUI remains
a no-op. The user closes the editor and applies through Litematica's configured
Execute Operation hotkey while Tool Mode 10 is active.

- Require Tool Mode 10 and a bound Execute Operation hotkey for editing.
- Expose the effective origin as a movable transient origin target.
- Moving it changes the world-axis offset and moves the preview immediately.
- Render the full proposed overlay, hide the old schematic content, and retain
  only the old bounds and baseline origin for context.
- Selecting a subregion or manual origin discards the transform draft.
- Leaving Tool Mode 10, changing projects/overlays, or unloading the world
  discards it.
- Shared hits prefer the focused target, then transform origin, then visible
  manual origin, then subregions alphabetically.

## Atomic Relocation

Execute applies rotation and offset together as one recoverable site-transform
operation. Apply the same preflight to rotation-only and offset-only changes:

- Require a clean Git worktree and manifest, no physical tracked differences,
  no competing draft, the correct dimension, and a full-fidelity integrated or
  Servux backend.
- Scan destination-only coverage for conflicting blocks, block entities, and
  entities.
- Report collision counts and sample positions, then allow an explicit
  overwrite confirmation.
- Warn that scheduled block/fluid ticks in the affected union will be cleared
  and cannot be preserved.
- Snapshot confirmed destination collisions in operation staging so rollback
  can restore them.
- Journal old/new rotation and offset plus operation phase.
- Freeze ticking, clear old coverage and collisions, restore canonical content
  at the draft transform, write both v2 fields, and rebuild the overlay.
- On failure or recovery rollback, restore the previous transform and collision
  snapshot. Recovery may instead resume the staged operation.

The completed transform remains an unsaved manifest change until Save Version.
Discard or switching to a revision with another transform physically relocates
the project through the same guarded workflow.

## Change Viewer

Translation is separate from rotation in presentation:

- Add a **Translation Offset** metadata row showing old and new vectors.
- Treat an absent offset as semantic zero for comparison. A change from
  Original to a non-zero rotation at `[0, 0, 0]` therefore produces only a
  Rotation row.
- Align comparison content to the target rotation and offset so translation
  itself never becomes raw block noise.
- Selecting Translation Offset highlights the baseline origin, effective
  origin, and current transformed bounds.

## Merge

Rotation and offset are coupled when branch orientations differ:

- If target and incoming rotations differ, keep the target rotation and target
  offset without a transform conflict.
- If rotations match, merge the offset with normal three-way rules.
- Keep equal values and one-sided changes.
- Raise a translation-offset conflict when both sides changed the offset
  differently.
- Resolve that conflict with the existing global Base/Incoming/Yours choice.
- Incoming-only sites retain their complete incoming transform.

## Export

The offset only positions Gitmatica tracking overlays and restored project
content. Do not write it into `.litematic` exports or custom export metadata.
Automatic and manual schematic origins naturally remove uniform world
translation; only the commit's rotation affects exported layout.

## Verification

- Cover missing/zero/non-zero offsets, direct non-zero rotation changes,
  returning to Original, negative values, and overflow.
- Verify baseline/effective-origin derivation across relogging, placement
  movement, Project Editor edits, and Tool Mode 10 movement.
- Cover draft rendering, cancellation, exclusive focus, hotkey gating, and
  combined Execute application.
- Cover collisions, confirmation cancellation, rollback snapshots, scheduled
  tick warnings, and recovery at every journal phase.
- Confirm offset-only changes preserve canonical hashes and create only the
  Translation Offset row.
- Cover same-rotation offset merges, two-sided conflicts, cross-rotation target
  preservation, and incoming-only sites.
- Verify all `.litematic` export paths omit the tracking offset.
