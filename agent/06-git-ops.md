# Git Operations

Gitmatica stores project history in normal Git repositories using JGit.

## Files

- `LvcGitHistoryOps`: commit listing and metadata.
- `LvcGitBranchOps`: create/delete/rename/checkout branch operations.
- `LvcGitBranchMetadata`: history-branch, branch-start, and remote-tracking
  metadata.
- `LvcGitRemoteOps`: remote URL, push, pull.
- `LvcGitUndoOps`: latest-version undo target and reset workflows.
- `LvcBranchMergeOps`: branch merge orchestration.
- `LvcSemanticMergeEngine`: merge orchestration and non-region manifest
  metadata.
- `LvcRegionMergeEngine`: name-based sub-region comparison, global conflict
  source selection, and merged chunk reconstruction.
- `LvcMergeObjectResolver`: object lookup during merges.
- `LvcGitTreeReader`: read files from commits/trees.
- `LvcGitHeadSnapshot`: branch/head DTO.

Use the focused owner directly. Do not reintroduce a project-wide Git facade or
duplicate its result records in a second namespace.

## Rules

- Keep semantic storage decisions out of generic Git helpers.
- Merge conflicts should be semantic/user-facing, not raw JGit exceptions.
- A sub-region's exact name is its identity; there is no separate region ID. A
  rename is delete plus add. Merge definitions first, then merge block payloads
  normally inside the resulting tracking masks.
- Preserve detached-head workflows; many UI actions operate at a selected commit.
- Soft load is still a real Git checkout: commit soft load may detach HEAD, while
  branch soft load attaches HEAD to the selected branch tip. It refreshes the
  tracking overlay but intentionally skips physical world restore.
- Treat a load as a no-op when the requested Git pointer and the current
  tracking overlay already match. Hard loads evaluate that identity before
  scheduling their unsaved-changes preflight; physical differences do not
  override an already-loaded target.
- Commit metadata should continue to identify the Minecraft player.
- SSH remotes are intentionally unsupported for the alpha build; remote UX
  should guide users toward HTTPS or local file remotes until explicit auth is
  designed.

## Commit Messages

For comprehensive Gitmatica changes, use the repository convention: a concise
imperative subject, a blank line, then `-` bullet points separated by blank
lines. Each bullet should summarize a distinct behavior or subsystem change.

## Docs

- `docs/tech/conflict-resolution.md`
- `docs/tech/lvc-semantic-storage.md`
