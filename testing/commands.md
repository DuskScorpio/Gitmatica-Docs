# Commands and task boundaries

Use the narrowest command that exercises the changed behavior. None of the
test tasks publish, sign, or deploy anything.

## Building without tests

The production command is:

```bash
./gradlew releaseJar
```

`releaseJar` assembles the installable addon JAR without running the test
suite. The development `jar` task also does not run tests.

Avoid `build` when the intention is only to produce a JAR. Gradle's `build`
lifecycle depends on `check`, and this project makes `check` depend on
`integrationTest`.

The `apply from:` statements in `build.gradle` only load task configuration
from files under `gradle/`. They do not execute those tasks.

## Fast development loop

Compile production code:

```bash
./gradlew agentCompile
```

Run the default integration lane:

```bash
./gradlew integrationTest
```

`agentTest` is an agent-friendly alias for the same integration lane:

```bash
./gradlew agentTest
```

Run one class or one method:

```bash
./gradlew integrationTest \
  --tests 'me.soullessjump.lvc.LvcBranchIntegrationTest'

./gradlew integrationTest \
  --tests 'me.soullessjump.lvc.LvcBranchIntegrationTest.headCommitDetectionComparesResolvedGitCommits'
```

Use `--rerun-tasks` when cached/up-to-date results are not acceptable:

```bash
./gradlew integrationTest --rerun-tasks
```

## Verification tasks

| Task | Runs or verifies |
| --- | --- |
| `classes` | Compiles production code. |
| `gametestClasses` | Compiles the separate GameTest mod and scenarios. |
| `integrationTest` | Runs 113 default JUnit tests; excludes `stress` and `performance` tags. |
| `stressTest` | Runs the three deterministic `stress`-tagged tests. |
| `performanceTest` | Runs the one `performance`-tagged 1,500-subchunk fixture. |
| `verifyReleaseJar` | Confirms harness/GameTest code and test-mod metadata are absent from the production JAR. |
| `verifyHarnessMetadata` | Checks translations, Mixin class targets, and addon asset boundaries. |
| `verifyHarnessSchemas` | Validates harness schemas, examples, and the exact request contract. |
| `verifyFast` | Compiles and runs the default semantic lane plus metadata checks. |
| `verifyAll` | Runs `check`, release isolation, metadata/schema checks, `doctor`, and GameTest compilation. |
| `doctor` | Writes machine-readable environment information to `build/harness/doctor.json`. |
| `collectEvidence` | Copies reports and harness artifacts into `build/evidence/`. |

Important boundaries:

- `verifyAll` compiles the Minecraft scenarios but does not launch Minecraft.
- `verifyAll` does not include `stressTest` or `performanceTest`.
- `integrationTest` does not include the stress or performance tagged tests.
- Minecraft and Servux scenarios only run through their explicit harness
  commands.

The comprehensive non-Minecraft command is therefore:

```bash
./gradlew verifyAll stressTest performanceTest
```

To rebuild every result and package the evidence:

```bash
./gradlew \
  verifyAll stressTest performanceTest collectEvidence \
  --rerun-tasks
```

## Minecraft client scenarios

Run the example boot request:

```bash
harness/run-client-gametest.sh \
  harness/examples/request.example.json \
  --accept-minecraft-eula
```

Run real project creation, schematic export/load, save, discard, checkout,
branch, and merge operations in disposable worlds:

```bash
harness/run-client-gametest.sh \
  harness/examples/project-lifecycle.request.example.json \
  --accept-minecraft-eula
```

To run another scenario, copy an example request, give it a new `runId`, set
the desired `scenarioIds`, and pass its repository-relative path to the
wrapper. Request paths should be relative to the project directory.

The lower-level equivalent is:

```bash
./gradlew runClientGameTest \
  -PacceptMinecraftEula=true \
  -PharnessRequest=harness/examples/request.example.json
```

The wrapper is preferred because it adds a hard timeout, headless Xvfb
support, and diagnostics for hung processes.

## Servux multiplayer

```bash
harness/run-servux-gametest.sh \
  harness/examples/servux.request.example.json \
  /path/to/reviewed-servux.jar \
  --accept-minecraft-eula
```

The controller starts an isolated localhost dedicated server, waits for
Minecraft and Servux readiness, runs a separate client GameTest process, then
stops the server process group.

## Reports

| Lane | Human-readable report | Machine-readable result |
| --- | --- | --- |
| Integration | `build/reports/tests/integrationTest/index.html` | `build/test-results/integrationTest/` |
| Stress | `build/reports/tests/stressTest/index.html` | `build/test-results/stressTest/` |
| Performance | `build/reports/tests/performanceTest/index.html` | `build/test-results/performanceTest/`, `build/performance/large-overlay.json` |
| Minecraft/Servux | Harness logs and screenshots | `build/harness/<runId>/result.json`, `events.jsonl` |

See [Evidence and failure triage](evidence-and-triage.md) before diagnosing a
Minecraft failure from console output alone.
