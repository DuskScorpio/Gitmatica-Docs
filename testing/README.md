# Gitmatica testing handbook

This directory is the operational source of truth for Gitmatica's automated
tests. It is written for maintainers and coding agents: start here, choose the
smallest lane that can disprove a change, and preserve structured evidence
when Minecraft is involved.

## Test layers

| Layer | What it proves | Primary command |
| --- | --- | --- |
| Static verification | Metadata, schemas, translations, Mixin targets, and release-JAR isolation are valid | `./gradlew verifyAll` |
| Integration | Semantic Git/LVC behavior works against temporary repositories and fixtures | `./gradlew integrationTest` |
| Stress and performance | Contention/lifecycle invariants hold and the large-overlay fixture records stable counters | `./gradlew stressTest performanceTest` |
| Minecraft client | The addon boots, opens a real UI, or enters a disposable singleplayer world | `harness/run-client-gametest.sh … --accept-minecraft-eula` |
| Servux multiplayer | A real disposable client and dedicated server negotiate Servux and select the Servux backend | `harness/run-servux-gametest.sh … --accept-minecraft-eula` |

There are 117 JUnit tests: 113 in the default integration lane, three tagged
stress tests, and one tagged performance test. The Minecraft harness has six
allow-listed scenarios.

## Quick start

Check the environment:

```bash
./gradlew doctor
```

Run all non-Minecraft checks, including the opt-in stress and performance
lanes:

```bash
./gradlew verifyAll stressTest performanceTest
```

Run the disposable Minecraft boot scenario:

```bash
harness/run-client-gametest.sh \
  harness/examples/request.example.json \
  --accept-minecraft-eula
```

Run the deterministic project lifecycle and merge scenarios:

```bash
harness/run-client-gametest.sh \
  harness/examples/project-lifecycle.request.example.json \
  --accept-minecraft-eula
```

Run the Servux scenario with a reviewed local Servux JAR:

```bash
harness/run-servux-gametest.sh \
  harness/examples/servux.request.example.json \
  build/harness/dependencies/servux-fabric-26.2-0.11.2.jar \
  --accept-minecraft-eula
```

Minecraft EULA acceptance is always explicit. The harness never uses the
developer's normal `run/` directory, existing saves, or a remote server.

## Where to go next

- [Commands and task boundaries](commands.md) explains exactly what every
  Gradle task and wrapper runs.
- [Complete test catalog](catalog.md) lists every JUnit test and Minecraft
  scenario by name and purpose.
- [Minecraft and Servux harness](minecraft-harness.md) documents requests,
  scenarios, fault injection, safety, and the two-process server lane.
- [Evidence and failure triage](evidence-and-triage.md) explains what to
  inspect after a failure and which files are authoritative.
- [Extending the suite](extending-tests.md) is the checklist for adding tests,
  scenarios, schemas, and CI coverage safely.

## Source map

| Concern | Authoritative source |
| --- | --- |
| JUnit task configuration | `gradle/testing.gradle` |
| Static and release checks | `gradle/verification.gradle` |
| Minecraft/Servux Gradle tasks | `gradle/harness.gradle` |
| JUnit sources | `src/integrationTest/` |
| Minecraft scenario sources | `src/gametest/` |
| Harness request and result schemas | `harness/schema/` |
| Local wrappers | `harness/run-client-gametest.sh`, `harness/run-servux-gametest.sh` |
| CI | `.github/workflows/ci.yml`, `.github/workflows/harness.yml` |

If this handbook and executable configuration disagree, the executable
configuration wins and this handbook should be updated in the same change.
