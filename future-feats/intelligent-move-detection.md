# Intelligent Move Detection

## Problem

Moving a large structure by only a few blocks creates thousands of coordinate
changes. The result is technically correct but difficult to understand in the
Change Viewer and world overlay.

## Planned Behavior

- Detect integer translations only. Rotation, mirroring, scaling, and arbitrary
  transformations are outside the first version.
- Match exact block states, including properties, with one source block assigned
  to at most one destination block.
- Use bounded, deterministic offset voting and verify only the strongest
  candidates.
- Require support, block-state diversity, local coverage, and dominance over
  overlapping alternatives.
- Automatically collapse only high-confidence moves. Present uncertain matches
  as suggestions that can be accepted or dismissed.
- Keep inventories and blocks added, removed, or changed after a move as raw
  residual differences.

## Presentation

- Add a **Moved** category to the Change Viewer.
- Show the signed `(x, y, z)` translation, exact matched-block count, and
  coverage.
- Provide **Show Raw** for every accepted move.
- Replace collapsed world-overlay noise with source and destination bounds
  connected by a movement vector.
- Highlight the matching world geometry when a move row is selected.

## Safety And Lifecycle

- Treat detection as viewer-only interpretation. Do not alter semantic storage,
  commits, restores, merges, or coordinate-exact verifier mismatches.
- Run analysis off the render thread from an immutable finished-verifier
  snapshot.
- Reject results whose verifier mismatch revision is stale.
- Rebuild only affected chunks and requeue a mesh if presentation changes while
  it is compiling.
- Recompute moves from the active verifier rather than persisting them as
  project or commit metadata.

Manual move definition is outside the first version. Ambiguous structures must
remain raw rather than being guessed.

This viewer-only heuristic is separate from
[rotation-aware project changes](rotation-aware-project-changes.md). Rotation
is an explicit versioned site transform that changes capture, restore, merge,
and overlay coordinate mapping; move detection only interprets ordinary
coordinate-exact block differences after they occur.

The prototype is isolated on `feat/move-detection` in commit `f34300a9`.
