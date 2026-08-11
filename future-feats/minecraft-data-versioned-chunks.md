# Minecraft DataVersion-Aware Chunks

## Status

TODO. The current `.lvcchunk` payload does not persist Minecraft `DataVersion`.
Do not add the field alone: safe support requires the payload conversion and
dirty-fingerprint rules below.

## Goal

Let one Gitmatica history safely retain immutable chunk objects captured by
different Minecraft versions. A commit may reference mixed-age objects, so the
version belongs to each chunk object rather than the project manifest.

## Format Evolution

- Introduce a new self-identifying chunk-content magic while retaining a reader
  for the current `LVCCHN3` content.
- Store the source Minecraft `DataVersion` in every new immutable chunk object.
- Include the chunk schema and `DataVersion` in the full object preimage so
  semantically different payload contracts cannot share an object ID.
- Let old and new objects coexist in Git history. Never rewrite historical
  commits or bulk-migrate objects merely because they were read.
- Define and document the assumed source semantics for legacy chunks that lack
  `DataVersion`; do not silently label them with the current version.

## Canonicalization And Data Fixing

- Decode each supported chunk schema into one current canonical runtime model.
- Apply Minecraft data fixes to block palettes, block-entity NBT, and entity NBT
  before current-version interpretation or mixed-version merge.
- Normalize both sides before semantic comparison and merge so one merged chunk
  never claims a single version while containing unfixed payloads from several
  versions.
- Reject unsupported future chunk schemas and meaningfully newer Minecraft data
  versions before restore, merge, or export writes anything.

## Dirty Fingerprints

- Give the verifier-visible tracked fingerprint its own explicit format/version
  instead of reusing the full object encoding.
- Keep storage-only schema and `DataVersion` changes out of the tracked
  fingerprint when the visible blocks and supported inventory projection are
  semantically equal.
- Preserve existing tracked hashes until a semantic recapture is required;
  introducing the new object writer must not mark every project chunk dirty.

## Rollout

1. Add golden fixtures and readers for the current and new chunk formats.
2. Add data-fix coverage for palettes, block entities, and entities across the
   supported Minecraft-version window.
3. Release read support before enabling the new writer by default.
4. Enable new writes for newly captured or semantically changed chunks while
   leaving unchanged legacy objects referenced in place.
5. Keep downgrade/export policy separate and require proof that a target format
   can represent the converted payload.

## Verification

- Decode current and new golden chunk fixtures into the expected canonical
  model and reject corrupt or unknown magic values.
- Load, scan, merge, restore, and export commits containing mixed-age objects.
- Verify data fixes for renamed blocks/properties and changed block-entity and
  entity schemas.
- Confirm a storage-only upgrade does not change tracked dirty state.
- Confirm unsupported future data fails before manifest, index, object, world,
  or export mutation.
