# Doc Maintenance

These docs are for agents, not users.

## Rules

- Prefer short files over long catch-all documents.
- Link to source owners instead of duplicating code logic.
- Document invariants and routing decisions.
- Do not copy method bodies or long algorithms.
- Update docs when an ownership boundary or durable rule changes.
- Delete stale notes quickly.
- Keep all future feature plans under `docs/future-feats/`.
- Give each future feature its own document. Other docs may link to that file,
  but must not duplicate its plan.

## Commit Messages

Match the repository's existing commit style:

- Imperative subject line, title-style wording, no trailing period.
- Blank line after the subject.
- Body as `- ` bullet points.
- Each bullet starts with a capitalized verb.
- Keep a blank line between bullets.

## Local Ignore

`AGENTS.md` and `docs/` are locally ignored through `.git/info/exclude` in this
workspace. If the user wants them tracked, remove those ignore entries first
and check `git status`.

## Existing Durable Docs

Do not duplicate these unless adding agent-only routing:

- `docs/tech/lvc-semantic-storage.md`
- `docs/tech/conflict-resolution.md`
- `docs/migration/architecture.md`
- `docs/testing/README.md`
