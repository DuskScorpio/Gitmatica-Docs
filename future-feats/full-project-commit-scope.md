# Full-Project Commit Scope

## Goal

Allow one version save to capture every site in a multi-site project instead of
being limited to the active site.

## Planned Behavior

- Resolve and validate every site placement before capture begins.
- Require authoritative access to every selected site's tracked chunks.
- Publish one atomic Git commit only after all selected sites capture
  successfully.
- Report unavailable sites as unknown and avoid silently creating a partial
  project version.
- If partial-site commits are exposed, make the scope explicit in the UI and
  commit summary.
- Preserve content-addressed object reuse and prune candidates only against the
  complete resulting manifest/index state.
- Show per-site progress and failure context during long captures.

The active-site workflow remains useful for focused work, but it must not be
presented as a complete project snapshot when other sites were omitted.
