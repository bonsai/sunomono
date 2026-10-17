# MVP and acceptance criteria

## Milestone 0 — technical validation
- [ ] Confirm OPFS availability in target MV3 execution context.
- [ ] Load SQLite WASM in a dedicated Worker.
- [ ] Create DB, insert/select a record, restart extension, and read it back.
- [ ] Record Chrome version, manifest permissions, Worker setup, VFS, and locking behavior.
- [ ] Document behavior when OPFS is unavailable.

## Milestone 1 — metadata core
- [ ] Apply versioned SQL migrations.
- [ ] Implement albums, tracks, assets, generation_runs, and events.
- [ ] Validate import/export JSON against versioned schemas.
- [ ] Provide create/read/update/search for tracks and albums.
- [ ] Record schema version and migration history.

## Milestone 2 — local Workspace
- [ ] Let the user select a Workspace directory through a supported browser API.
- [ ] Re-check saved handle permission before use.
- [ ] Store relative artifact paths, not assumed absolute paths.
- [ ] Check file existence, size, MIME type, and SHA-256 when requested.
- [ ] Report missing files, permission loss, path escape, and hash mismatch.
- [ ] Provide metadata export/import and a documented recovery path.

## Acceptance tests
1. Persistence: a test record survives extension restart.
2. Migration: previous schema upgrades without losing fixture records.
3. Export/import: round-trip reproduces supported metadata.
4. Workspace permission: missing/revoked permission does not crash or access files silently.
5. Integrity: missing assets and checksum mismatch are reported.
6. Conflict: overwrites are detected and require explicit confirmation.
7. Failure safety: failed initialization does not delete DB or claim data is saved.
8. Regression: no manifest, service-worker, command, or storage-key collisions.

## Out of scope
Cloud sync, automatic destructive cleanup, automatic recording without user action, large-scale migration before inventory and rollback approval.
