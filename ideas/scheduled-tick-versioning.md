# Scheduled Tick Versioning

## Status

Unconfirmed idea. Gitmatica currently does not store scheduled ticks, and this
document does not commit the project to supporting them.

## Problem

Litematic files can store `PendingBlockTicks` and `PendingFluidTicks`. A record
contains a target registry ID, subregion-relative position, remaining `Time`,
`Priority`, and `SubTick` ordering value.

These values are runtime scheduler state rather than stable build data:

- `Time` counts down and may disappear when the tick executes.
- `SubTick` orders equal-time, equal-priority work within one scheduler
  timeline. Values from separate branches cannot be compared meaningfully.
- Tick order can affect timing-sensitive redstone, pistons, and fluids.

Consequently, combining ticks from multiple branches cannot be made correct in
all cases.

## Candidate Commit Behavior

- Snapshot scheduled block and fluid ticks as they exist when the user saves a
  version.
- Store each tick once per site position, independent of overlapping
  subregions.
- Keep ticks in full commit payloads but out of tracked dirty fingerprints and
  the Change Viewer.
- Preserve the stored snapshot for export and restore without interpreting what
  the scheduled operation is intended to accomplish.
- Capture only through an authoritative server scheduler. Multiplayer support
  would require corresponding Servux protocol support.

## Candidate Merge Policy

Treat the checked-out branch's pre-merge `HEAD` as the only tick source:

1. Begin with the ticks stored in Current `HEAD`.
2. Merge blocks and fluids normally.
3. Retain a Current tick only when its position remains tracked, its target is
   valid, and the final merged block state or fluid still matches Current
   `HEAD`.
4. Strip the tick when Incoming or Base supplies a different final state.
5. Never import Incoming ticks or resurrect Base ticks.

If a Base or Incoming choice happens to produce the same final state as Current
`HEAD`, the Current tick may remain; its source is still Current. A block-entity
or inventory-only incoming change does not remove an otherwise compatible
block tick.

There are no tick conflicts, tick-specific dialogs, or Change Viewer rows. The
user implicitly recreates missing ticks by activating the imported machinery
in the merged world before a later save. Replacing a block alone may not be
enough; for example, a pressure plate needs an entity to activate its polling
tick.

## Ordering

All retained ticks come from one `HEAD` scheduler snapshot, so merging never
combines unrelated `SubTick` namespaces. Removing records leaves their relative
order intact and does not require contiguous values.

Restore should clear scheduled activity in the affected tracked area, apply
blocks, then enqueue the retained ticks while preserving their `Time`,
`Priority`, and relative `SubTick` order. If the runtime scheduler must assign
fresh ordering values, sort retained records deterministically before enqueueing
them rather than comparing against Incoming or Base values.

## Known Tradeoffs

- The policy deliberately discards all Incoming and historical Base scheduler
  state, even when it might have mattered to the build.
- A saved tick snapshot can become stale as soon as the world continues
  running.
- Restoring `Time` reproduces the saved remaining delay, not elapsed real time.
- Existing destination-world ticks in the restored area must be replaced, not
  mixed with the snapshot.
- This covers only scheduled block and fluid ticks. It does not preserve random
  ticks, block-event queues, neighbor-update queues, or general entity ticking.
- Exact behavior cannot be guaranteed; this is a deterministic, Current-biased
  policy rather than a semantic merge of running simulations.

## Open Decisions

- Whether tick-only changes should allow a new version when tracked blocks are
  otherwise unchanged.
- Whether ordinary project initialization should capture live ticks and whether
  direct Litematic import should preserve embedded ticks.
- Whether unsupported multiplayer capture/restore should omit ticks with a
  warning or make tick preservation unavailable for that operation.
- Whether the added storage and backend complexity is worthwhile at all.
