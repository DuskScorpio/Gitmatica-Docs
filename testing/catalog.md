# Complete test catalog

This catalog mirrors `src/integrationTest/` and `src/gametest/`. It is intended
to answer two questions quickly:

1. Which existing test is closest to the behavior being changed?
2. Which invariants are already covered and must remain true?

## Default integration lane

The 113 tests below run under `integrationTest` and therefore also under
`check` and `verifyAll`.

### `LvcVerifierHiddenMismatchIntegrationTest` — 3

- `hiddenStateRestores` — hidden mismatch state survives restoration.
- `inventoryMetadataUsesIdentity` — inventory mismatch metadata uses stable
  entry identity.
- `renderFiltersAreRevisioned` — verifier render filters advance their
  revision when changed.

### `LvcBranchIntegrationTest` — 9

- `commitHistoryRemainsScopedToBranchAfterCheckout`
- `headCommitDetectionComparesResolvedGitCommits`
- `headPointerNameShowsBranchOrDetachedCommit`
- `detachedHeadAtBranchTipCanReattachWithoutRestore`
- `detachedOlderHeadCanReattachAfterCheckingOutBranchTip`
- `createBranchShowsBranchStartAndLaterCommits`
- `createBranchRejectsDuplicateInvalidAndEmptyRepoInputs`
- `deleteBranchRemovesNonCurrentBranchAndRejectsProtectedInputs`
- `renameBranchUpdatesRefsMetadataAndRejectsInvalidInputs`

These cover branch-scoped history, attached/detached HEAD presentation and
reattachment, plus branch create/delete/rename validation.

### `LvcCheckoutUndoIntegrationTest` — 9

- `checkoutUpdatesSemanticWorkingTreeWhilePreservingVisibleHistory`
- `sameTipBranchCheckoutPreservesDirtyTrackedFiles`
- `commitAfterCheckoutIsRejectedWhileHeadIsDetached`
- `resetWorkingTreeToHeadDiscardsTrackedDirtyChanges`
- `undoLatestCommitKeepChangesPreservesDirtyWorkingTree`
- `saveVersionAfterKeepChangesUndoRecommitsDirtyWorkingTree`
- `undoLatestCommitDeleteChangesHardResetsToParent`
- `checkoutBranchAndResetToCommitMovesBranchTip`
- `checkoutCanContinueAfterResettingDirtyWorkingTree`

These cover semantic checkout, dirty-tree handling, detached commit guards,
both undo modes, branch-tip movement, and checkout recovery.

### `LvcFriendlyErrorsIntegrationTest` — 5

- `friendlyErrorsMapTypedUnloadedChunks`
- `friendlyErrorsMapWrappedTypedCheckoutPreflightUnloadedChunks`
- `friendlyErrorsUseInterruptedMessageForRecoverySensitiveRestoreFailures`
- `friendlyErrorsMapTypedUnreadableCaptureChunks`
- `friendlyErrorsHideUnexpectedRawDetails`

These assert that typed and wrapped failures become useful user messages
without leaking inappropriate raw error details.

### `LvcMergeIntegrationTest` — 5

- `mergeBranchFastForwardsCurrentBranch`
- `mergeBranchCreatesSemanticTwoParentMergeCommit`
- `mergeBranchCancelsSemanticConflictsWithoutMovingHead`
- `mergeBranchConflictsWhenSameInventoryChangesDifferently`
- `mergeBranchConflictResolutionAcceptsBaseIncomingAndYours`

These cover fast-forward and two-parent merges, conflict cancellation,
inventory conflicts, and all supported semantic conflict choices.

### `LvcOperationRecoveryIntegrationTest` — 13

- `faultInjectionStaysInertOutsideDisposableGameTest`
- `operationJournalWritesPrimaryBackupAndNoTemp`
- `operationJournalRejectsChecksumCorruption`
- `operationJournalCanBeWrittenBeforeGitDirectoryExists`
- `operationJournalIgnoresTempOnlyCrashLeftovers`
- `operationJournalReadsPrimaryWhenBackupWasNeverCopied`
- `operationJournalFallsBackToBackupWhenPrimaryIsMissing`
- `operationJournalFallsBackToBackupWhenPrimaryIsCorrupt`
- `operationJournalQuarantinesCorruptRecoveryData`
- `operationJournalDeleteClearsActiveBackupTempAndLegacyLocations`
- `interruptedStagingCleanupPreservesPublishedObjects`
- `refreshMarkerSurvivesUntilOverlayCleanup`
- `transientOverlayRemovalRecordsRefreshIntent`

These protect crash recovery, atomic journal replacement, checksum and backup
fallback behavior, object staging cleanup, and overlay refresh intent.

### `LvcProjectRepositoryIntegrationTest` — 7

- `semanticRepoCommitsUseCurrentBranchHeadAndHistoryListsNewestFirst`
- `projectServiceListsSemanticRepositoriesAndPushesToRemote`
- `projectRepositoryNamesCannotEscapeProjectRoot`
- `projectSuccessPathsStartAtMinecraftDirectory`
- `projectServiceDeletesSemanticRepositoriesRecursively`
- `remoteUrlConfigCanBeCreatedAndEdited`
- `pushUsesLastActiveBranchWhileHeadIsDetached`

These cover project discovery, repository containment, path presentation,
deletion, remote configuration/push, and detached-HEAD branch selection.

### `LvcRemoteSparsePlannerIntegrationTest` — 5

- `remoteServuxSparsePlannerTracksVerifierVisibleBlockEntityInventory`
- `remoteServuxSparsePlannerRequiresAuthoritativeInventoryAfterReconnect`
- `remoteCommandSparsePlannerBuildsBlockStateOnlySchematic`
- `remoteCommandSparsePlannerSkipsUntrackedGaps`
- `remoteCommandSparsePlannerRejectsUnreadableBlocks`

These cover sparse planning for Servux and command backends, authoritative
inventory freshness, untracked gaps, and unreadable data.

### `LvcSemanticCaptureScanIntegrationTest` — 10

- `captureStoresGapsAsUntrackedPositions`
- `captureChangesOnlyTheIntersectingStorageChunkHash`
- `captureAppliesLocalSiteOriginBeforeReadingWorld`
- `semanticScanHashesWithoutWritingObjects`
- `semanticScanDetectsBlockEntityInventoryChanges`
- `semanticScanIgnoresEntityOnlyChanges`
- `semanticRestoreModesHandleBlockEntityOnlyChangesByOperation`
- `semanticScanComparesTrackedHashesNotObjectReferences`
- `semanticScanPreflightCanIgnoreStaleStoredTrackedHashes`
- `semanticScanReportsUnavailableChunksAsUnknown`

These cover world capture, local origins, chunk-level change isolation,
read-only scanning, block-entity inventories, entity exclusion, restore
modes, stale hash preflight, and unavailable chunks.

### `LvcSemanticChunkStorageIntegrationTest` — 12

- `semanticChunkEncodingIsDeterministicAndRoundTrips`
- `semanticChunkStorageWritesRawPayloads`
- `semanticChunkRejectsDuplicateBlockEntityRecords`
- `semanticChunkDecodeRejectsTruncatedPayloads`
- `semanticChunkStoreWritesContentAddressedObjectsOnce`
- `semanticHashIndexCodecStoresFullAndTrackedHashes`
- `minecraftBlockStateStringsAreCanonical`
- `canonicalBlockEntityNbtSortsKeysAndIgnoresPosition`
- `semanticEntityNbtStripsRuntimeEntityIds`
- `servuxBulkEntityNbtBecomesProjectRelative`
- `trackedChunkHashIncludesVerifierVisibleBlockEntityInventory`
- `trackedChunkHashNormalizesStoredAirVariants`

These protect deterministic encoding, strict decoding, content-addressed
storage, canonical state/NBT forms, Servux coordinate conversion, and tracked
hash semantics.

### `LvcSemanticManifestIntegrationTest` — 7

- `semanticManifestAndLocalStateRoundTrip`
- `semanticManifestAllowsOverlappingRegionsAsTrackedUnionMasks`
- `projectServiceMapsSelectionToSemanticSite`
- `projectServiceUpdatesRegionsFromExistingPlacementOrigin`
- `trackingOverlayFirstLoadUsesSeededProjectOrigin`
- `projectServiceCreatesEmptyBrowserProjectWithoutCommit`
- `projectSummaryToleratesMissingLocalStateWithoutCreatingIt`

These cover manifest/local-state persistence, overlapping regions, selection
mapping, placement origins, initial overlays, empty projects, and read-only
summary behavior.

### `LvcSemanticOverlayCacheIntegrationTest` — 5

- `currentOverlayCacheRejectsParentPreviewDescriptors`
- `trackingOverlayRevisionTargetsCurrentParentAndRootAir`
- `trackingOverlayDisplayNameIncludesShortHeadCommit`
- `semanticTrackingOverlayCacheIsFileBacked`
- `corruptSemanticTrackingOverlayDescriptorIsRecoverable`

These cover current/parent overlay identity, root-air previews, display names,
file-backed caching, and corrupt descriptor recovery.

### `LvcSemanticRepositoryLifecycleIntegrationTest` — 8

- `semanticRepositoryInitCommitsManifestAndObjects`
- `semanticRepositoryNoOpCommitReportsNoChanges`
- `semanticLossyCommitEmptiesPreviousBlockEntities`
- `semanticRepositoryCommitUpdatesOnlyChangedChunkReference`
- `semanticRepositoryCommitPrunesReplacedObjectFromCurrentHead`
- `semanticRepositoryCommitKeepsSharedLiveObjectCandidate`
- `semanticObjectPrunerAbortsBeforeDeletingWhenNewObjectIsMissing`
- `semanticRepositoryUpdateAreasChangesRegionsAndChunkRefs`

These cover initialization, no-op commits, lossy changes, reference updates,
safe pruning, shared objects, and area changes.

### `LvcSemanticSchematicIntegrationTest` — 6

- `semanticAirSchematicPreservesCommitRegions`
- `semanticWorkingTreeSchematicLoadsFullHashes`
- `semanticSparseSchematicUsesStructureVoidForSkippedBlocks`
- `semanticWorkingTreeSchematicLoadsBlockEntityNbt`
- `semanticWorkingTreeSchematicPromotesContainerComponents`
- `semanticCommitExportUsesSelectedCommit`

These cover air/root schematics, full and sparse materialization, block
entities and container components, and commit export selection.

### `LvcTaskRegistryIntegrationTest` — 6

- `overlayLoadingBlocksForegroundOperations`
- `foregroundOperationsBlockOverlayLoading`
- `diagnosticSnapshotReportsOperationOwnership`
- `worldUnloadClearsOperationOwnership`
- `worldUnloadInvalidatesCallbacksFromPreviousTaskEpoch`
- `specializedTasksParticipateInWorldUnloadFencing`

These cover mutual exclusion, diagnostic ownership, world-unload cleanup,
epoch fencing, and specialized task registration.

### `LvcSpatialDiffGroupsIntegrationTest` — 3

- `sixDirectionConnectivity`
- `diagonalSeparationAndOrder`
- `filteredEntryProjection`

These cover change-group connectivity, deterministic separation/order, and
filtered projections.

## Opt-in JUnit lanes

These four tests are deliberately excluded from `integrationTest`.

### `LvcHarnessStressIntegrationTest` — 3 (`stressTest`)

- `operationRegistryRemainsCleanUnderContention`
- `worldUnloadFencingRemainsStable`
- `atomicMetadataReplacementNeverExposesPartialContents`

They repeatedly exercise task contention, unload fencing, and atomic
replacement. Keep them deterministic; a stress tag is not permission for a
flaky test.

### `LvcLargeOverlayPerformanceTest` — 1 (`performanceTest`)

- `largeOverlayFixtureRecordsBuildCounters`

It builds a stable 1,500-subchunk fixture and records counters in
`build/performance/large-overlay.json`. It is a regression signal, not a
machine-independent microbenchmark with a hard wall-clock threshold.

## Static verification

`verifyHarnessMetadata` verifies:

- `zh_cn` and `zh_tw` contain the same translation keys as `en_us`.
- every declared Mixin class exists.
- the addon does not ship replacement `assets/litematica` resources.

`verifyHarnessSchemas` parses the harness schemas and examples and enforces
the exact request contract.

`verifyReleaseJar` asserts that the installable JAR excludes:

- `me/soullessjump/lvc/gametest/`
- `me/soullessjump/lvc/harness/`
- the `gitmatica-gametest` test-mod marker

## Disposable Minecraft scenarios

The scenario registry is a compiled allow-list; arbitrary class names or
commands cannot be requested.

### `boot-state-snapshot`

- Audits expected Mixin application when requested.
- Verifies required mods.
- Captures client and Gitmatica semantic state.
- Saves a screenshot and structured diagnostics.

Use it for dependency, entrypoint, Mixin, and basic initialization changes.

### `ui-project-browser`

- Opens `GuiLvcProjectBrowser`.
- Verifies that the expected screen is active.
- Captures diagnostics and a screenshot.
- Returns to `TitleScreen`.

Use it for screen construction, navigation, and render smoke coverage.

### `singleplayer-world-smoke`

- Creates a disposable empty superflat world.
- Waits for the client world, player, and required chunks.
- Captures state and a screenshot.
- Disconnects cleanly.

Use it when behavior requires an integrated server or real world lifecycle.

### `singleplayer-project-lifecycle`

- Seeds an exact generated block fixture in a disposable integrated-server
  world.
- Creates a project through the production initialization task and verifies
  its first commit, branch, tracking overlay, and clean semantic scan.
- Changes the world and saves a second version through the production commit
  task.
- Exports both commits, reloads them through Litematica, and checks every
  block in both schematic containers.
- Makes unsaved changes, runs the production discard task, and verifies the
  world returned exactly to the saved fixture.
- Checks out the initial commit and then the main tip through production
  preflight/apply tasks, checking world blocks, HEAD state, overlay identity,
  and cleanup.

Use it for changes to project initialization, world capture, saving,
schematic export/load, discard, checkout, overlay rebuilding, task locking,
or operation cleanup.

### `singleplayer-merge-lifecycle`

- Creates a real base project and a feature branch.
- Captures one world change on the feature branch and a disjoint world change
  on main.
- Runs the production semantic merge and restore path.
- Verifies the exact merged world, branch and HEAD, two-parent ordering,
  commit count, clean scan, tracking overlay, and absence of journal/marker
  residue.

Use it for branch capture, merge planning, semantic merge commits, merge
world restore, merge overlay refresh, and merge cleanup.

### `servux-multiplayer-smoke`

- Is accepted only by the two-process Servux controller.
- Enables Gitmatica's entity-data sync and Servux paste path before joining.
- Connects to the controller's fixed localhost port.
- Waits for the real Servux handshake.
- Asserts that Gitmatica selected `LvcWorldBackend.SERVUX`.
- Captures evidence, disconnects, and returns to the title screen.

Use it for client/server negotiation, backend selection, and Servux-specific
behavior. It does not yet prove every Gitmatica operation over Servux; add a
focused scenario when an operation needs end-to-end coverage.
