# Component-Aware Inventory Comparison

## Problem

Inventory comparison currently considers empty state, item type, and count. It
can miss changes to custom names, lore, damage, enchantments, nested container
data, and other item components.

## Planned Behavior

- Compare the complete component state of corresponding `ItemStack` values in
  addition to item type and count.
- Treat a component-only change as a changed slot.
- Detect changes to `DataComponents.CONTAINER` so a modified shulker-box item is
  visible even when its outer item type and count are unchanged.
- Include components in open-container refresh signatures so packet updates
  cannot leave a component-only difference stale.
- Keep comparison snapshots immutable and independent of later screen or world
  mutations.

Tech Utils' optional `ItemStack.getComponents()` comparison is a useful
reference. This feature changes classification of the outer slot only; a
recursive UI for inspecting nested slots is planned separately.

## Verification

- Cover names, lore, damage, enchantments, and container components.
- Verify both singleplayer and authoritative Servux snapshots.
- Confirm that component-only packet refreshes update an already open preview.
