# Semantic Data

## Versioned Project Files

Current semantic format is centered on:

- `lvc.json`: manifest.
- `indexes/*.lvcidx`: hash indexes.
- `objects/sha256/**.lvcchunk`: immutable chunk objects.

Use `docs/tech/lvc-semantic-storage.md` as the durable format reference.

## Main Types

- `LvcManifest`: current canonical project/sites/regions/index model. It does
  not contain storage schema provenance or codec settings.
- `LvcManifestDocument` pairs a canonical manifest with the source/write
  `LvcManifestSchema` at the storage boundary.
- `LvcManifest.Site`: one placed coordinate space, including its
  project-relative manual origin. V1 reads as zero; the manifest store upgrades
  a v1 write to v2 when a nonzero origin first requires it.
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
- `storage.manifest`: version-specific manifest wire DTOs, schema selection,
  and atomic manifest I/O.
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

Manifest reads decode every supported schema into the canonical model without
rewriting history. Existing writes preserve their schema floor, upgrade to the
minimum schema required by the result, and never downgrade implicitly. New
manifests use the current schema. Merge results preserve the highest schema of
their base, current, and incoming documents.
