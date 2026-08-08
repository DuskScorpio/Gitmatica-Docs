# Evidence and failure triage

Start with structured outputs, then use prose logs to explain them. This avoids
chasing harmless Minecraft warnings while missing the actual scenario
failure.

## Evidence locations

### JUnit

| Lane | HTML | XML |
| --- | --- | --- |
| Integration | `build/reports/tests/integrationTest/index.html` | `build/test-results/integrationTest/` |
| Stress | `build/reports/tests/stressTest/index.html` | `build/test-results/stressTest/` |
| Performance | `build/reports/tests/performanceTest/index.html` | `build/test-results/performanceTest/` |

The performance fixture also writes:

```text
build/performance/large-overlay.json
```

### Environment

`./gradlew doctor` writes:

```text
build/harness/doctor.json
```

It records the Git revision/dirty state, Java and Gradle versions, Minecraft
dependency versions, OS/architecture, Xvfb availability, free disk, run-root
safety information, and whether EULA acceptance was supplied to that Gradle
invocation.

### Minecraft and Servux

For run ID `<runId>`:

```text
build/harness/<runId>/
```

Inspect:

1. `result.json` — authoritative pass/fail and final scenario summary.
2. `events.jsonl` — ordered scenario phases and the first structured failure.
3. `diagnostics/runtime-events.jsonl` — Gitmatica operation ownership,
   lifecycle, and runtime diagnostics.
4. `heartbeat.json` — last known liveness.
5. `screenshots/` — visible state at scenario checkpoints.
6. `logs/` — Minecraft/client/server prose logs and controller diagnostics.

`collectEvidence` packages available reports, results, harness outputs,
performance counters, selected logs, crash reports, screenshots, and Mixin
diagnostics under:

```text
build/evidence/
```

CI uploads this directory with a 14-day retention period.

## Triage order

### 1. Identify the failed layer

- Gradle configuration/compilation failure: no tests ran; inspect the first
  compiler or task-configuration error.
- JUnit failure: open the lane's HTML report and reproduce the narrow class or
  method.
- Harness validation failure: inspect the request, EULA flag, path, schema,
  run ID, and scenario allow-list.
- Minecraft scenario failure: start with `result.json` and `events.jsonl`.
- Hung Minecraft process: inspect thread dumps, process-tree capture,
  heartbeat age, and the last runtime event.
- Servux failure: distinguish server startup/readiness from client connection,
  handshake, and backend-selection failures.

### 2. Reproduce narrowly

For one JUnit method:

```bash
./gradlew integrationTest \
  --tests 'fully.qualified.ClassName.methodName' \
  --rerun-tasks
```

For Minecraft, copy the request, retain only the failing scenario, and choose
a fresh run ID. Do not delete the original evidence before comparing it.

### 3. Determine whether the failure is deterministic

Run the same narrow case again with a new harness run ID. If it is
intermittent, compare:

- scenario event order;
- operation/task epochs and ownership;
- world unload/reload timing;
- overlay publication and refresh markers;
- Servux handshake timing;
- process/thread dumps for blocked work.

Do not fix a race by adding an arbitrary sleep. Add an observable readiness
condition or lifecycle invariant and a regression test.

### 4. Verify the fix at increasing scope

1. failing method or scenario;
2. containing JUnit lane or related scenario set;
3. `verifyAll stressTest performanceTest`;
4. relevant Minecraft/Servux scenario;
5. `collectEvidence` when handing off or running in CI.

## Common harness validation failures

| Symptom | Check |
| --- | --- |
| EULA rejection | Pass `--accept-minecraft-eula` to the wrapper or `-PacceptMinecraftEula=true` to Gradle. |
| Request not found | Use a repository-relative request path and verify the file exists. |
| Unsafe `runId` | Use 1–96 letters, digits, `.`, `_`, or `-`, beginning with a letter or digit. |
| Unknown scenario | Use an ID listed in [the catalog](catalog.md#disposable-minecraft-scenarios). |
| Existing/mixed evidence | Create a new immutable run ID. |
| Servux JAR rejection | Supply an existing `.jar` containing valid Servux Fabric metadata. |
| No Servux handshake | Check server readiness, client feature flags, protocol compatibility, and runtime events. |

## Known non-fatal disposable-run noise

The following messages have appeared in successful disposable runs:

- an illegal anisotropic-filtering value of `0`;
- missing `build/run/clientGameTest/config/malilib.json` on a fresh run;
- Realms authorization or profile-key `401` responses for the synthetic
  `Player0`;
- block-state migration warnings involving a `false` wall property.

These are observations, not a permanent ignore list. They are non-fatal only
when the structured scenario result passes and no new related failure appears.
Re-evaluate them if versions change or the warning correlates with a failing
assertion.

## Handoff checklist

When reporting a test result, include:

- exact command;
- Git commit and whether the tree was dirty (`doctor.json`);
- number of passed/failed/skipped tests;
- harness run ID and scenario IDs, if applicable;
- first actionable failure with its evidence path;
- whether Servux was used and which reviewed JAR/digest was supplied;
- any test-related code or fixture changes made during diagnosis.
