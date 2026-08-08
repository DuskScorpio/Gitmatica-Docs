# Standalone Addon Architecture

Gitmatica is a client-side addon for the released Minecraft 26.2 versions of
Litematica and MaLiLib. The initial addon migration was made from
`refactor/major-cleanup` at commit
`3001d7dc6163a42fd42f9e83bb35bfa19a615a8c`.

The addon remains single-version while its core behavior is being stabilized.
The gates and branch policy for later backports are in
`docs/migration/version-support.md`.

## Boundaries

- `me.soullessjump.lvc` owns project data, Git operations, capture, restore, tasks,
  overlays, and Gitmatica screens.
- `integration.litematica` is the public adapter layer for Litematica and
  MaLiLib registrations.
- `mixin` contains narrow hooks into behavior that Litematica does not expose.
  Domain code must not cast to a mixin class; it uses an addon-owned interface.
- `gui.widgets` owns reusable Gitmatica UI components. Screens coordinate
  workflows and layout rather than reimplementing widgets.
- `assets/gitmatica` contains every Gitmatica resource. The addon must never
  replace `assets/litematica` files.

## Runtime Invariants

- A tracking overlay's position is its Litematica placement origin.
- Checkout, save, discard, reload, and close never move an overlay to the
  player.
- The stale rebuild guard verifies placement identity immediately before a
  rebuilt schematic chunk is published.
- LVC task cancellation uses snapshot scheduler iteration, so cancellation
  cannot skip or corrupt another scheduled task.
- Inventory verifier data and filtered render state are sidecars owned by
  Gitmatica; they do not extend Litematica enums.
- The semantic overlay cache remains a derived cache. Git history and semantic
  chunk objects remain canonical.

## Deliberately Excluded Fork Changes

Generic Litematica changes that were not required by Gitmatica were not carried
into the addon. This includes legacy Schematica conversion changes, Litematica
resource replacements, renderer enum additions, and unrelated GPU sampler
changes.

## Build and Debug

Use Java 25. `Ctrl+Shift+B` runs the production build in VS Code. The Run and
Debug panel contains normal and mixin-diagnostic Minecraft client launches.

```sh
env JAVA_HOME=/home/arnav/.jdks/temurin-25.0.3 ./gradlew build
env JAVA_HOME=/home/arnav/.jdks/temurin-25.0.3 ./gradlew integrationTest
```
