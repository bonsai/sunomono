# sunomono

Suno production workspace integrating a Manifest V3 Chrome extension (CRX), browser-local SQLite WASM/OPFS, and a separate local production workspace.

## Architecture principles

- **GitHub is the source of truth for specifications:** MRD, ADRs, JSON Schemas, SQL migrations, tests, and implementation.
- **SQLite WASM + OPFS** stores browser-side production metadata, indexes, workflow state, and events.
- **The local Workspace** stores audio files and other production artifacts. It is separate from OPFS.
- **The CRX** provides browser UI and user-initiated actions.
- **Local agents/CLI** handle file processing, conversion, validation, and GitHub workflows.

## First milestone

Validate SQLite WASM/OPFS inside the target MV3 extension context before building the full integrated UI. Verify persistence after extension restart, Worker/WASM loading, locking behavior, and backup/restore.

## Documents

- [MRD and MVP](docs/MRD.md)
- [Architecture decision record: storage boundaries](docs/adr/0001-storage-boundaries.md)
- [Data contract](docs/data-contract.md)
- [MVP acceptance criteria](docs/MVP.md)

## Safety and data handling

- Do not assume OPFS is a normal Windows folder; it is origin-scoped browser storage.
- Access a local Workspace only after explicit user folder selection and permission.
- Do not silently fall back to non-persistent storage when OPFS fails.
- Do not delete, move, or overwrite existing Suno repositories or local files as part of initial integration.
- Audio capture/export must remain explicit user actions.

## Related planning

See [bonsai/crxman#41](https://github.com/bonsai/crxman/issues/41) and the integration inventory issues #48–#51 in that repository.
