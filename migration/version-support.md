# Version Support and Backport Policy

Gitmatica will use one source branch per supported Minecraft/Litematica/MaLiLib
compatibility line. It will not open those branches while the product and its
internal contracts are still changing substantially.

## Current Phase

Development stays on `main` and targets the single released dependency set in
`gradle.properties` (currently Minecraft 26.2). There is no promise of
backports during this phase.

Prepare for later ports without maintaining them yet:

- Keep Minecraft, Litematica, and MaLiLib calls in `integration/litematica` or
  narrow mixins wherever practical.
- Keep domain records, Git operations, and semantic storage independent of
  Minecraft classes unless the type is genuinely part of their responsibility.
- Version persisted semantic formats and test migrations before changing them.
- Keep platform-adapter changes separate from domain changes when practical, so
  later cherry-picks have small conflict surfaces.
- Test addon-owned integration interfaces and the core workflows that use them.

Do not add speculative compatibility abstractions or a source preprocessor
before a real port demonstrates that they are needed.

## Stabilization Gate

Open the first compatibility branch only after:

- Project create, open, clone, and delete workflows are reliable.
- Capture, save, commit, checkout, discard, branch, history, and remote
  workflows have settled.
- Merge/conflict behavior and the semantic storage format are stable, or their
  migrations are tested.
- Overlay origin, rebuild, close, multiplayer/Servux apply, cancellation, and
  recovery behavior satisfy the runtime invariants.
- The major screen navigation and addon-owned Litematica interfaces are no
  longer being reorganized.
- At least two consecutive releases are predominantly fixes and refinements
  rather than cross-cutting feature or architecture changes.
- The production build and integration suite are green.

Explicitly deferred features do not block a port. Remaining work does block it
when it is expected to redesign storage, integration contracts, or most core
workflows.

## Branch Model

When the gate is met:

1. Tag the common stable commit used as the first port base.
2. Keep `main` as the primary compatibility line and create
   `mc/<minecraft-version>` branches for maintained alternatives.
3. Pin one exact Minecraft, Litematica, MaLiLib, loader, mappings, Java, and
   build-tool set on each branch.
4. Port one line at a time, in user-demand order, and prefer the final or most
   relevant patch release in a Minecraft family.
5. Build, run integration tests, and audit required mixins independently on
   every maintained branch.
6. Publish artifacts with their Minecraft compatibility line in the name and
   metadata.

Use ordinary, unconditional Java on each branch. Reconsider Stonecutter or
another source-sharing system only if Gitmatica later needs near-feature parity
across many simultaneously maintained versions and measured cherry-pick effort
has become the dominant maintenance cost.

## Support Tiers

- **Primary (`main`)**: New features, architecture work, and all fixes.
- **Maintained (`mc/*`)**: Bug and data-compatibility fixes, plus feature
  backports that are small and low risk.
- **Legacy**: Corruption, crash, security, and critical interoperability fixes
  only; no feature parity promise.

Document each released line's tier and dependencies. A line may be retired
instead of accepting a fix whose safe implementation requires a large
architecture fork.

## Change Flow

- A compatibility-neutral defect shared by maintained lines should normally be
  fixed on the oldest affected line and carried forward with
  `git cherry-pick -x`.
- A platform-specific defect should be fixed on its owning branch and copied
  only where the same platform behavior exists.
- A large feature lands on `main`; older-line backports are optional.
- A storage-format or migration fix goes to every supported version that can
  read or write the affected data.
- Backports preserve behavior and data compatibility; they do not silently
  import unrelated refactors.

Branching is a release-maintenance policy, not a reason to duplicate domain
logic. The integration boundary remains the preferred place for unavoidable
version differences.
