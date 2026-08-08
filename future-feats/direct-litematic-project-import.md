# Direct Litematic Project Import

## Goal

Create a semantic Gitmatica project directly from a `.litematic` without first
pasting it and converting its placement into an area selection.

## Required Semantics

- Preserve named sub-regions and their project-relative bounds.
- Preserve tracked gaps between independent sub-regions instead of importing
  them as air.
- Seed the Gitmatica placement from the imported schematic origin through normal
  Litematica placement ownership.
- Canonicalize block states, block entities, inventories, and supported entities
  through the existing semantic storage codecs.
- Reject an import when overlapping sub-regions contain conflicting payloads for
  the same project coordinate. Never choose one payload implicitly.
- Report unsupported or unreadable payloads before publishing the repository.

## Open Decisions

- Whether import begins from Project Browser, Loaded Schematics, or both.
- Whether the first commit keeps the source schematic name and metadata.
- How duplicate sub-region names are surfaced and resolved before capture.
