# Minecraft and Servux harness

The harness is a separate Fabric GameTest mod compiled from `src/gametest/`.
It gives agents structured observability while keeping automation endpoints
out of the production addon.

## Safety model

The harness may only:

- execute scenario IDs from a compiled allow-list;
- use generated fixtures under `build/run/`;
- connect the Servux client to a controller-owned localhost port;
- write evidence under `build/harness/` and `build/evidence/`.

It must never:

- open a normal developer save under `run/`;
- connect to a remote address;
- accept shell commands, Java class names, or arbitrary output paths from a
  request;
- package harness endpoints into `releaseJar`;
- publish artifacts or modify golden baselines without human review.

Client GameTests require `-PacceptMinecraftEula=true`. The wrapper exposes
that acknowledgement as `--accept-minecraft-eula`.

## Request format

The authoritative schema is `harness/schema/request.schema.json`. Start from
an example in `harness/examples/`.

A request has:

- `schemaVersion`: currently `1`;
- `runId`: a unique immutable identifier matching
  `[A-Za-z0-9][A-Za-z0-9._-]{0,95}`;
- `scenarioIds`: a non-empty list of allow-listed scenarios;
- scenario options defined by the schema, such as required-mod or Mixin
  checks.

Use a new `runId` for every invocation. Reusing a completed run directory can
mix evidence and defeats the one-request/one-result model.

Pass request files by repository-relative path. The Gradle validation task
resolves them against the project directory.

## Client controller

Preferred command:

```bash
harness/run-client-gametest.sh \
  path/to/request.json \
  --accept-minecraft-eula
```

The controller:

1. validates the EULA acknowledgement and request;
2. launches Fabric's disposable client GameTest run;
3. uses Xvfb when no display is available;
4. imposes a hard outer timeout;
5. captures a process tree and Java thread dump before terminating a hung
   process;
6. leaves structured evidence in `build/harness/<runId>/`.

Inside Minecraft, the driver writes a heartbeat every two seconds and
enforces a best-effort scenario deadline. The outer controller is the final
defense against a process that cannot make in-process progress.

## Servux controller

Preferred command:

```bash
harness/run-servux-gametest.sh \
  path/to/servux-request.json \
  /path/to/reviewed-servux.jar \
  --accept-minecraft-eula
```

The controller:

1. validates that the supplied file is a JAR with Servux Fabric metadata;
2. prepares the isolated server directory at
   `build/run/servux-server`;
3. binds a dedicated Minecraft server to localhost with one player slot;
4. waits for both Minecraft server readiness and Servux readiness;
5. starts a separate disposable client process on the chosen local port;
6. waits for the real client/server handshake and scenario result;
7. terminates the complete server process group during cleanup.

The client scenario must configure Gitmatica features needed for the
handshake before connecting. The current smoke scenario enables entity-data
sync and Servux pasting, then proves `LvcWorldBackend.SERVUX` was selected.

The controller accepts no hostname. CI may download Servux only from a
maintainer-reviewed HTTPS URL and must verify its configured exact digest
before executing it.

## Structured runtime protocol

Each run directory has this shape:

```text
build/harness/<runId>/
├── request.json
├── events.jsonl
├── result.json
├── heartbeat.json
├── logs/
├── screenshots/
└── diagnostics/
    └── runtime-events.jsonl
```

`events.jsonl` records scenario lifecycle events matching
`harness/schema/event.schema.json`. Sequence numbers are monotonic within
that stream.

`diagnostics/runtime-events.jsonl` uses the same event shape for Gitmatica's
operation/task diagnostics, but it deliberately has a separate sequence
counter.

`result.json`, matching `harness/schema/result.schema.json`, is the
authoritative final scenario result. Do not infer success solely from a
screenshot or the absence of a dramatic console error.

## Fault injection

Fault injection exists to prove operation recovery across precisely placed
crash boundaries. It is inert unless both the separate GameTest mod and the
disposable-run acknowledgement are present.

Allowed fault points:

- `journal_written`
- `object_staged`
- `git_head_moved`
- `world_block_applied`
- `refresh_marker_written`
- `overlay_published`
- `authoritative_sync_scheduled`

Supported modes:

- `exception` — throw at the point and let normal process teardown occur;
- `halt` — terminate abruptly to simulate a crash boundary.

Low-level invocation:

```bash
./gradlew runClientGameTest \
  -PacceptMinecraftEula=true \
  -PharnessRequest=harness/examples/request.example.json \
  -PharnessFaultPoint=journal_written \
  -PharnessFaultMode=exception
```

Use a dedicated request/run ID for every injected failure. A useful recovery
test is normally a pair: one run creates the interrupted state and a second
clean run verifies the intended recovery behavior.

## Scenario selection guide

| Change | Minimum useful scenario |
| --- | --- |
| Dependency, entrypoint, Mixin, or startup state | `boot-state-snapshot` |
| Screen construction or basic UI navigation | `ui-project-browser` |
| Integrated-server lifecycle, chunks, or world access | `singleplayer-world-smoke` |
| Project init, save, Litematica export/load, discard, or checkout | `singleplayer-project-lifecycle` |
| Branch capture, semantic merge, merge restore, or merge parents | `singleplayer-merge-lifecycle` |
| Servux handshake or remote world backend | `servux-multiplayer-smoke` |
| Pure semantic Git, storage, or scan logic | Prefer an integration test; no Minecraft launch needed |

The project scenarios invoke production task implementations directly through
a test-only driver. They bypass GUI clicks but do not bypass operation locks,
world capture/restore, journals, epochs, Git operations, or Litematica overlay
creation. The driver and scenarios are excluded from `releaseJar`.

The UI scenario is a smoke test, not pixel-perfect visual regression.
Screenshots are diagnostic evidence and should be inspected when relevant.
