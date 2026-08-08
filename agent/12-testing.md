# Testing

## Commands

Compile:

```sh
env JAVA_HOME=/home/arnav/.jdks/temurin-25.0.3 ./gradlew compileJava compileIntegrationTestJava processResources
```

Integration tests:

```sh
env JAVA_HOME=/home/arnav/.jdks/temurin-25.0.3 ./gradlew integrationTest
```

Whitespace:

```sh
git diff --check
```

## Integration Test Files

- `LvcRepositoryIntegrationTest`: main integration entrypoint.
- `LvcSemanticStorageIntegrationTest`: semantic storage, overlay, export, scan.
- `LvcOperationRecoveryIntegrationTest`: journal/recovery.
- `LvcFriendlyErrorsIntegrationTest`: user-facing errors.
- `IntegrationTestSupport`, `LvcIntegrationFixtures`, `FakeWorldReader`:
  helpers and fixtures.

## Test Scope Rules

- Do not add tests for very small naming/display-only changes unless the user
  asks or the change has meaningful behavioral risk.
- Storage codec/hash changes need deterministic round-trip tests.
- Git operation changes need branch/history tests.
- Operation-journal changes need corruption/fallback tests.
- UI-only changes can often be compile-only, but workflow wiring should get an
  integration test when it affects service/task behavior.
- Litematica async/renderer issues usually still need manual Minecraft testing.

## VS Code Hot Reload

- Use `Debug Minecraft Client` or `Debug Minecraft Client (Mixins)`. Their
  pre-launch task prepares Loom and copies the resolved Sponge Mixin Java agent
  to `.gradle/vscode/mixin-agent.jar`.
- The Java debugger's built-in workspace build stays disabled because it runs
  before `preLaunchTask` and can observe partially imported Gradle source sets.
  The hot-reload pre-launch task instead runs Gradle's `classes` task, rebuilds
  `bin/main` from the compiled main classes and processed resources, and then
  validates that every main Java source has top-level bytecode and no class
  contains Eclipse's unresolved-compilation stub. A failed rebuild or
  validation must stop the launch.
- Saving a Java edit compiles and applies Hot Code Replace automatically. The
  Microsoft Gradle BSP importer is disabled because its HCR-triggered compile
  completion creates a build feedback loop; VS Code uses the traditional
  Buildship Gradle importer instead. The hot-reload profiles use the IntelliJ
  JBR 25 managed by JetBrains Toolbox with enhanced class redefinition enabled.
- Treat method-body edits as the reliable hot-reload path. Restart the client
  for new or removed fields, nested types, class-shape changes, static
  initialization changes, or any reload that leaves runtime state inconsistent.
- Prioritize the simplest maintainable production design over hot-reload
  compatibility. Never add indirection, helper classes, or alternate ownership
  solely to avoid a client restart; tell the user that the change requires one.
- Mixin reload support comes from the Java agent. Some changes can still be too
  invasive for a live JVM or leave existing runtime state stale; restart the
  client when a reload is rejected or behavior does not match the new code.
- Reload client assets with `F3+T` and server data with `/reload` after Gradle
  has processed the changed resources.
