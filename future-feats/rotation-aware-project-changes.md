# Rotation-Aware Project Changes

## Goal

Allow branches to develop the same canonical build in different horizontal
orientations. Rotation is a versioned project change, not thousands of moved,
added, and removed blocks. Canonical edits can therefore merge across rotated
branches and appear in the target branch's orientation.

The linked [translation-offset plan](rotated-overlay-translation-offsets.md)
defines how a rotated overlay is repositioned relative to its unrotated fixed
origin and owns the combined in-game transform editor.

## Canonical Model

Store every site in one unrotated project coordinate frame. For a site without
a translation offset:

```text
world position = placement origin + rotate(canonical position)
canonical position = inverseRotate(world position - placement origin)
```

Support only rotation around Minecraft's Y axis:

- Original / 0 degrees.
- 90 degrees clockwise.
- 180 degrees.
- 90 degrees counterclockwise.

Mirroring, vertical rotation, and arbitrary transforms are outside the first
version. Rotation uses Gitmatica's fixed project origin as its pivot; manual
origin remains a separate versioned coordinate used by its marker and export.

Canonical subregion bounds, chunk coordinates, block states, block entities,
entities, and manual-origin coordinates remain unchanged by a pure rotation.
Content hashes must therefore remain stable.

## Manifest V2

Rotation is part of the unreleased `lvc-manifest-v2`; do not introduce v3.
Add a per-site `rotation` field using stable Minecraft-compatible names.

- Missing rotation in prerelease v2 data means Original / 0 degrees.
- V1 data also reads as Original / 0 degrees without rewriting history.
- New v2 writes include the site's rotation.
- The optional `rotation_offset` and its validity rules belong to the linked
  translation-offset plan.

## Capture And Materialization

- Forward-transform canonical positions and states for overlays, verification,
  restore, checkout, discard, undo, and world application.
- Inverse-transform world positions and directional block states during scan
  and capture.
- Normalize transformed subregion corners back into positive-size canonical
  bounds after Tool Mode 10 edits.
- Use Minecraft's `BlockState.rotate` behavior for directional blocks.
- Match Litematica's entity rotation behavior for positions, yaw, and passenger
  trees.
- Carry block-entity NBT normally. Do not attempt to rewrite arbitrary
  coordinates embedded in vanilla or modded NBT.
- Reject arithmetic overflow before mutation.
- Permit physical rotation only through full-fidelity integrated-server or
  Servux backends; reject command-only fallback.

Tracking overlays may use canonical schematic content with their authoritative
site rotation applied at placement time. Gitmatica must block normal Litematica
main-placement and subregion rotation/mirror controls for tracking overlays;
rotation is changed through Project Editor. Local placement-origin movement
remains available and must not dirty the repository.

## Apply And Recovery

Rotation and translation use one atomic site-transform workflow defined in the
linked offset plan. Before applying a rotation, require a clean repository, no
physical tracked differences, no pending edit draft, the correct dimension,
and no active operation.

The operation must be journaled and crash recoverable. It freezes affected
ticking, clears old coverage, applies canonical content at the new transform,
publishes the v2 manifest change, and rebuilds the overlay. Scheduled block and
fluid ticks are not stored; warn that affected ticks will be cleared before the
user confirms.

Checkout, branch switching, discard, undo, and revision restore must use the
same orientation-aware relocation path whenever source and target rotations
differ.

## Change Viewer

- Add a distinct **Rotation** metadata row showing old and new orientations.
- Align physical comparison data to the target/displayed orientation so a pure
  rotation produces no raw block rows.
- Compare working changes to HEAD in the working orientation.
- Compare a historical commit to its parent in the selected commit's
  orientation.
- Selecting Rotation highlights the current transformed bounds and project
  pivot.
- Do not add an exact-orientation preview toggle in the first version.

Real canonical block, inventory, entity, and subregion changes continue to use
their normal Change Viewer rows.

## Merge

Merge subregions and content entirely in canonical coordinates.

- Existing sites keep the target/current branch's rotation when source and
  target rotations differ.
- Incoming-only sites retain their incoming rotation and offset.
- Incoming rotation never creates a content conflict or replaces the target
  orientation.
- Blocks that both branches changed at the same canonical position still use
  normal semantic conflict rules.
- Do not fast-forward when doing so would replace an existing target site's
  transform; create a semantic merge commit instead.
- Same-transform histories may still fast-forward normally.

Rotation-aware cherry-picking is a later feature. The first version makes the
existing semantic branch merge rotation-independent.

## Export

Export the selected commit in its stored rotation:

- Rotate region bounds, block states, and entities.
- Automatic origin uses the rotated minimum bounds.
- Manual-origin export uses the rotated canonical manual origin.
- Do not encode Gitmatica's tracking-overlay translation offset in the
  `.litematic`; export placement is a separate concern.

## Verification

- Cover all four rotations, inverse/composition rules, negative coordinates,
  bounds normalization, and overflow rejection.
- Round-trip representative directional blocks, including pistons, observers,
  repeaters, rails, stairs, chests, hoppers, and redstone components.
- Verify Litematica-compatible entity/passenger rotation and unchanged
  arbitrary block-entity NBT.
- Confirm pure rotation preserves canonical hashes and scans cleanly.
- Cover prerelease v2 fields, v1 defaults, historical reads, and v2 round trips.
- Cover rotated overlays, relog restoration, manual origins, subregion edits,
  full-fidelity backend gating, checkout/discard/undo, and recovery.
- Merge differently rotated branches and verify the target orientation remains
  unchanged while canonical edits land at the correct transformed positions.
- Verify rotated automatic/manual-origin exports.
