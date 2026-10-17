# ADR-0001: Storage boundaries

- Status: Proposed; validate with a technical proof of concept
- Date: 2026-10-17

## Context
sunomono needs browser UI, persistent metadata, and local audio artifacts. OPFS is browser-managed origin-private storage, not the user's normal Windows directory. SQLite WASM OPFS support has Worker/VFS and locking considerations. A local folder requires a supported user-mediated access flow.

## Decision proposal
1. Keep specifications, schemas, migrations, source, and tests in GitHub.
2. Use SQLite WASM + OPFS for browser-side metadata, indexes, workflow status, and event history.
3. Keep audio and other large artifacts in a separate user-selected local Workspace.
4. Access SQLite through a dedicated Worker; validate the exact MV3 context and VFS in a PoC.
5. Use explicit folder selection and permission checks for Workspace access. If unsupported, offer JSON export/download instead of assuming arbitrary local filesystem access.
6. Do not require an always-on localhost bridge for the first MVP.
7. Keep metadata export/import as a backup and migration path.
8. Do not put audio blobs in SQLite in the MVP.

## Consequences
Positive: clear metadata/artifact separation; browser UI does not require a permanently running local process; schema changes are reviewable; backup can be tested independently.

Risks: OPFS is origin-scoped; VFS and multi-tab locking need empirical verification; folder permission can be revoked; browser metadata and local files can become inconsistent.

## Alternatives
- Local SQLite only: rejected for the requested browser-local WASM/OPFS design.
- Audio in SQLite BLOBs: deferred due to database growth and backup cost.
- Mandatory localhost bridge: deferred until a demonstrated use case justifies operational/security cost.
- Git as the live production database: rejected; Git is the versioned specification/code source, not runtime DB.

## Validation required before Accepted
Demonstrate DB creation and persistence from the target extension context; test restart and multi-tab behavior; verify Worker/WASM packaging and CSP; test export/import and recovery; record SQLite WASM build and VFS.
