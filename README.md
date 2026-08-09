# Gitmatica documentation

This directory keeps maintainer and agent documentation close to the code.

- [Testing handbook](testing/README.md) — commands, complete test catalog,
  Minecraft/Servux automation, evidence triage, and extension rules.
- [Addon migration architecture](migration/architecture.md) — dependency and
  ownership boundaries for the standalone Gitmatica addon.

For test work, always begin with the testing handbook. It identifies the
smallest relevant lane and the authoritative evidence produced by that lane.

## Future Feature Plans

Future plans live only in `future-feats/`, with one feature per document:

- [Intelligent move detection](future-feats/intelligent-move-detection.md)
- [Component-aware inventory comparison](future-feats/component-aware-inventory-comparison.md)
- [Nested container inventory preview](future-feats/nested-container-inventory-preview.md)
- [Scalable hash indexes](future-feats/scalable-hash-indexes.md)
- [Direct Litematic project import](future-feats/direct-litematic-project-import.md)
- [Manual origin export](future-feats/manual-origin-export.md)
- [Consistent operation preflights](future-feats/consistent-operation-preflights.md)
- [Full-project commit scope](future-feats/full-project-commit-scope.md)
