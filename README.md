# Gitmatica documentation

This directory keeps maintainer and agent documentation close to the code.

- [Testing handbook](testing/README.md) — commands, complete test catalog,
  Minecraft/Servux automation, evidence triage, and extension rules.
- [Addon migration architecture](migration/architecture.md) — dependency and
  ownership boundaries for the standalone Gitmatica addon.

For test work, always begin with the testing handbook. It identifies the
smallest relevant lane and the authoritative evidence produced by that lane.

## Future Feature TODO

This is the canonical future-feature index. Update the status here whenever a
plan is added, implemented, reduced to follow-up work, or removed. Detailed
plans live only in `future-feats/`, with one feature per document.

Planned:

- [Intelligent move detection](future-feats/intelligent-move-detection.md)
- [Component-aware inventory comparison](future-feats/component-aware-inventory-comparison.md)
- [Nested container inventory preview](future-feats/nested-container-inventory-preview.md)
- [Direct Litematic project import](future-feats/direct-litematic-project-import.md)
- [Version export flow](future-feats/export-flow.md)
- [Consistent operation preflights](future-feats/consistent-operation-preflights.md)
- [Full-project commit scope](future-feats/full-project-commit-scope.md)
- [Minecraft DataVersion-aware chunks](future-feats/minecraft-data-versioned-chunks.md)

Conditional on measurement:

- [Scalable hash indexes](future-feats/scalable-hash-indexes.md)

Implemented with deferred follow-ups:

- [Manual origin](future-feats/manual-origin.md) — Export integration and
  pending-draft mutation blocking remain.
- [Subregion diff visualization](future-feats/sub-region-diff-visualization.md)
  — Coincident-outline presentation remains.
