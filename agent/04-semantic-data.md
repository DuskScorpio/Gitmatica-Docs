# Semantic Data

## Versioned Project Files

Current semantic format is centered on:

- `lvc.json`: manifest.
- `indexes/*.lvcidx`: hash indexes.
- `objects/sha256/**.lvcchunk`: immutable chunk objects.

Use `docs/tech/lvc-semantic-storage.md` as the durable format reference.

## Main Types

- `LvcManifest`: project/sites/regions/index refs.
- `LvcManifest.Site`: one placed coordinate space, including its versioned
  project-relative manual origin. V1 reads as zero; an origin edit upgrades the
  working manifest to v2.
- `LvcManifest.Region`: tracked region inside a site.
- `LvcSitePlacement`: dimension plus origin DTO.
- `LvcChunk`: canonical block-state/block-entity/entity payload.
- `LvcChunkCoordinate`: chunk address.
- `LvcIntPosition`: compact position record.

## Hash Semantics

- Full chunk hashes store the full block-state, block-entity, and entity payload
  needed for restore/export.
- Tracked chunk hashes are for user-visible dirty detection. They include
  canonical block states plus verifier-visible inventory projection only:
  slot, item id, and count from legacy `Items` or component-backed container
  data.
- Tracked hashes intentionally ignore arbitrary block entity NBT such as custom
  names, furnace timers, loot table metadata, and item stack components unless
  verifier UI is expanded to explain those differences.
- Entity payloads are full-hash only and do not dirty tracked hashes.

## Storage Owners

- `LvcSemanticRepository`: writes/reads manifests, indexes, commits.
- `LvcRepository`: lower-level repo helpers.
- `LvcChunkCodec`: deterministic chunk codec.
- `LvcHashIndexCodec`: full/tracked hash index codec.
- `LvcChunkStore`: content-addressed object store.
- `LvcChunkStagingStore`: operation-local staging.
- `LvcSemanticObjectPruner`: removes replaced unreferenced objects.
- `LvcCanonicalNbt`: canonicalizes NBT for stable hashing.

Do not hand-edit versioned semantic files with ad hoc strings. Use these owners.

Manual-origin changes are repository metadata. They participate in dirty,
commit, discard, checkout, undo, and three-way merge behavior, but must not
affect tracked chunk hashes, overlay identity, verification, or Change Viewer
entries.
