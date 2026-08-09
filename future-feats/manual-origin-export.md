# Manual Origin Export

## Scope

Manual origin only affects schematic export.

## Stored Data

- Store whether manual origin is on or off.
- If on, store the manual origin position relative to project origin.
- If off, no manual origin position is stored.

## Coordinates

- Stored positions are project-relative.
- The UI may show and edit the position as world coordinates by adding or
  subtracting the current project placement origin.
- World coordinates are not stored.

## Export

- Off: export uses the default schematic origin, likely bounds min XYZ.
- On: export uses the stored manual origin.

## UI

Follow Litematica behavior:

- Off: the manual origin editor is hidden.
- On:
  - the editor is shown;
  - the user can type world XYZ;
  - the user can move the origin to the player;
  - the user can reset the origin.

## Reset

Reset restores the latest commit's schematic origin.

- If the latest commit had manual origin on, reset uses the committed manual
  origin.
- If the latest commit had manual origin off, reset uses that commit's default
  schematic origin, likely bounds min XYZ.
- Current uncommitted bounds do not affect reset.

## Merge

- Resolve on/off first.
- If the final state is off, ignore manual origin position conflicts.
- If the final state is on, resolve manual origin position conflicts.
