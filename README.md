# Gitmatica documentation

This repository keeps maintainer and agent documentation for
[Gitmatica](https://github.com/soullessjump/gitmatica), a Litematica addon
that adds Git-backed version control for Minecraft builds.

It is a preserved mirror of the original
[`soullessjump/Gitmatica-Docs`](https://github.com/soullessjump/Gitmatica-Docs)
repository, which is private and no longer maintained. The original
documentation was authored by **Arnavpmr** (`soullessjump`); this copy exists so
the docs stay available to the community.

## Contents

- [Testing handbook](testing/README.md) — commands, complete test catalog,
  Minecraft/Servux automation, evidence triage, and extension rules.
- [Addon migration architecture](migration/architecture.md) — dependency and
  ownership boundaries for the standalone Gitmatica addon.
- [Agent guide](agent/00-start-here.md) — routing table, invariants, and source
  owners for coding agents working on the addon.
- [Future feature plans](future-feats/) — one document per planned feature.

For test work, always begin with the testing handbook. It identifies the
smallest relevant lane and the authoritative evidence produced by that lane.

## Status

The docs describe the addon as of August 2026. They are kept as a historical
reference; new development happens in the
[main Gitmatica repository](https://github.com/soullessjump/gitmatica).

## License

Documentation is released under the [MIT License](LICENSE).
