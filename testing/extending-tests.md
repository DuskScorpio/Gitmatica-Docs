# Extending the test suite

Add coverage at the lowest layer that can faithfully exercise the behavior.
Pure semantic logic belongs in JUnit; use Minecraft only when a real client,
screen, world, integrated server, or network handshake is essential.

## Add a default integration test

1. Place the test in `src/integrationTest/java/`.
2. Reuse temporary repositories, fixtures, and helpers already used by the
   nearest cataloged class.
3. Name the method as a behavioral contract, not an implementation detail.
4. Control time, randomness, paths, and thread completion explicitly.
5. Assert both the result and important negative side effects, such as an
   unchanged HEAD, no staged object leak, or no task ownership left behind.
6. Run the method, its class, then `integrationTest`.
7. Add the method to [the complete catalog](catalog.md).

Prefer a regression test that fails for the actual bug before changing
production code.

## Add stress coverage

Tag a deterministic JUnit test with the suite's `stress` tag and place it near
other lifecycle/contention tests.

A stress test should:

- exercise a precise invariant many times or under controlled contention;
- use bounded waits and report enough state to diagnose timeouts;
- clean up executors, tasks, repositories, and temporary state;
- never depend on “it probably races within N milliseconds.”

Run:

```bash
./gradlew stressTest --rerun-tasks
```

## Add performance coverage

Tag the test `performance` and record stable work counters or allocations that
can be compared across implementations. Avoid a hard timing assertion unless
the execution environment is controlled well enough to make it meaningful.

Run:

```bash
./gradlew performanceTest --rerun-tasks
```

If a new artifact is produced, include it in `collectEvidence` and document
its schema and interpretation.

## Add a Minecraft scenario

A scenario is a public harness contract. Updating only the Java scenario is
not sufficient.

1. Implement it under `src/gametest/`.
2. Register a stable ID in the compiled scenario registry.
3. Define an explicit readiness condition and bounded deadline.
4. Capture structured diagnostic checkpoints and a screenshot when visual
   state matters.
5. Restore a safe screen/world state and release resources on success and
   failure.
6. Update `harness/schema/request.schema.json` if options change.
7. Add or update a request example.
8. Update `verifyHarnessSchemas` when the exact request contract changes.
9. Add the scenario to [the catalog](catalog.md).
10. Decide whether CI should run it automatically, manually, or only when an
    external reviewed dependency is configured.

Never add reflection targets, shell fragments, arbitrary server addresses, or
arbitrary output paths to the request format.

## UI scenario guidance

For a screen test:

- enter through the same public navigation path a player uses when practical;
- verify the screen class and critical enabled/disabled state;
- use a deterministic window size and fixture state;
- wait for an observable screen condition rather than sleeping;
- capture a screenshot after the screen is stable;
- close back to a real safe screen such as `TitleScreen`, not `null`.

Keep visual styling assertions focused. A screenshot is useful evidence, but
a smoke scenario should not become a brittle pixel comparison unless a
reviewed golden-image workflow is intentionally introduced.

## Singleplayer scenario guidance

- Generate the world entirely under the disposable task run directory.
- Use the test-only project driver when the behavior maps to an existing
  production task; do not reimplement capture, restore, Git, or overlay logic
  in the scenario.
- Prefer empty superflat or another minimal deterministic generator.
- Wait for the client world, player, integrated server, and required chunks.
- Assert the exact world fixture, semantic scan state, Git pointer/parents,
  overlay identity/origin, and operation residue that matter to the workflow.
- Do not open or copy a developer's existing save.
- Disconnect and wait for teardown before completing the scenario.
- Record task ownership, world epoch, and operation state around lifecycle
  transitions when testing races.

## Servux scenario guidance

- Use only the two-process localhost controller.
- Supply and verify a reviewed Servux JAR.
- Enable required Gitmatica protocol features before connection.
- Separate assertions for server readiness, connection, Servux handshake, and
  Gitmatica backend selection.
- Add operation-specific assertions when testing more than negotiation.
- Ensure server cleanup covers success, assertion failure, timeout, and
  interruption.

## Add a fault point

Fault points are security-sensitive and must remain inert in production.

1. Place the hook at a durable operation boundary with a clear before/after
   invariant.
2. Add its ID to the Gradle allow-list and the runtime allow-list.
3. Require the separate GameTest mod plus disposable-run acknowledgement.
4. Cover both injected interruption and subsequent recovery.
5. Test `exception` first; add `halt` only when abrupt process death is
   necessary.
6. Document the point in
   [Minecraft and Servux harness](minecraft-harness.md#fault-injection).
7. Confirm `verifyReleaseJar` still proves harness code is absent.

## CI and evidence checklist

Before considering new coverage complete:

- the narrow new test passes with `--rerun-tasks`;
- the containing lane passes;
- `verifyHarnessMetadata` and `verifyHarnessSchemas` pass when relevant;
- `verifyReleaseJar` confirms test code is not shipped;
- new reports or logs are included by `collectEvidence`;
- CI has an explicit timeout;
- external downloads require a reviewed URL and exact digest;
- EULA acceptance remains explicit;
- no workflow publishes, signs, accesses a real world/server, or changes a
  baseline without human approval;
- this handbook and the catalog are updated.

## Review rule

Tests are part of the feature design. Review test-only helpers with the same
care as production code: especially path containment, thread cleanup,
operation fencing, stable identities, recovery artifacts, and any boundary
between the disposable harness and the production addon.
