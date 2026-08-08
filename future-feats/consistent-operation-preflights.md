# Consistent Operation Preflights

## Goal

Use one authoritative scan result model before every operation whose safety
depends on the current world matching project state.

## Planned Behavior

- Reuse the same clean, dirty, stale, unknown, and mismatch-sample semantics for
  commit, checkout, pull, reset, merge, discard, and clear workflows.
- Never report clean when any required chunk or site lacks authoritative data.
- Keep backend limitations explicit for integrated-server, Servux, and command
  fallback modes.
- Present actionable mismatch samples without requiring each workflow to invent
  its own scan UI.
- Share scan work when consecutive workflow stages target the same immutable
  world revision, while invalidating it after relevant block or inventory
  changes.
- Keep cancellation and world-unload fencing consistent with other long-running
  Gitmatica tasks.

Existing workflows that already perform authoritative preflight checks should
converge on this shared contract rather than being replaced wholesale.
