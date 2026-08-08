# Start Here

Use these docs as a routing table, not as a replacement for source.

## Workflow

1. Identify the subsystem from `01-source-map.md`.
2. Read that subsystem note and the source files it names.
3. Make the smallest source change that respects `02-invariants.md`.
4. Run the narrow compile/test command first, then broader tests when behavior
   crosses package boundaries.
5. Update these docs only when a durable rule, flow, or ownership boundary
   changes.

## Upstream Bug Gate

- When evidence shows a bug is entirely outside Gitmatica, stop after diagnosis.
- Summarize the responsible upstream component, the evidence, user impact, and
  any plausible upstream or compatibility fix.
- Ask the user before implementing a Gitmatica workaround or compatibility shim.
- Continue directly only when some part of the bug is owned by Gitmatica; keep
  that fix narrowly separated from any optional upstream workaround.

## High-Value Source Files

- Project catalog: `src/main/java/me/soullessjump/lvc/project/LvcProjectCatalog.java`
- Semantic operations: `src/main/java/me/soullessjump/lvc/semantic/LvcSemanticProjectOperations.java`
- Overlay lifecycle: `src/main/java/me/soullessjump/lvc/overlay/LvcTrackingOverlayService.java`
- Semantic repository: `src/main/java/me/soullessjump/lvc/storage/LvcSemanticRepository.java`
- Long operations: `src/main/java/me/soullessjump/lvc/task/`
- Future feature plans: `docs/future-feats/`
- Main Gitmatica screen: `src/main/java/me/soullessjump/lvc/gui/GuiLvcProjectManager.java`
- Project editor: `src/main/java/me/soullessjump/lvc/gui/GuiLvcProjectEditor.java`
- Litematica placement-manager reference source:
  `codebases/litematica/src/main/java/fi/dy/masa/litematica/schematic/placement/`

## Always Check

- Is this Gitmatica code or upstream Litematica behavior?
- Does this touch placement origin?
- Does this run in singleplayer, multiplayer with Servux, or both?
- Does it need operation-journal crash recovery?
- Is this UI consistent with Litematica/MaLiLib patterns?
