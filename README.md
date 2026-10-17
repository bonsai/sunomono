# sunomono

Suno production workspace integrating a Manifest V3 Chrome extension (CRX), browser-local SQLite WASM/OPFS, a WS Bridge, and a separate local production workspace.

## Architecture principles

- **GitHub is the source of truth for prompts and specifications:** prompt files, MRD, ADRs, JSON Schemas, SQL migrations, tests, and implementation.
- **GH API + Local Agent** retrieves and validates prompts, creates run IDs, and dispatches one generation job at a time.
- **WS Bridge** transports versioned commands/events between the local agent and CRX. Actual endpoints and method names must be confirmed against the bridge implementation before coding.
- **The CRX** provides browser UI and submits prompts through the Suno Web UI, with validated commands and explicit user control.
- **SQLite WASM + OPFS** stores browser-side production metadata, indexes, workflow state, and events.
- **The local Workspace** stores recorded audio and other production artifacts. It is separate from OPFS.
- **Recorder** records the resulting track only after an explicit user action.

## Target workflow

GitHub prompt/spec → GH API → Local Agent → WS Bridge → CRX → Suno Web → identify generated clip → user-initiated recording → Local Workspace → SQLite metadata/index → export result manifest.

See [WS Bridge workflow](docs/ws-bridge-workflow.md) for sequence, state machine, correlation IDs, reliability, and security.

## First milestones

1. Validate SQLite WASM/OPFS inside the target MV3 extension context.
2. Inspect the existing WS Bridge and Recorder implementation and document actual methods, endpoints, approvals, and permissions.
3. Freeze the data/message contracts before implementing the end-to-end flow.
4. Implement one prompt/run at a time with idempotency and observable states.

## Documents

- [MRD](docs/MRD.md)
- [MVP and acceptance criteria](docs/MVP.md)
- [Architecture](docs/architecture.md)
- [Storage boundaries ADR](docs/adr/0001-storage-boundaries.md)
- [Data contract](docs/data-contract.md)
- [WS Bridge workflow](docs/ws-bridge-workflow.md)

## Safety and data handling

- OPFS is origin-scoped browser storage, not a normal Windows folder.
- Access a local Workspace only after explicit user folder selection and permission.
- Do not silently fall back to non-persistent storage when OPFS fails.
- Do not delete, move, or overwrite existing Suno repositories or local files as part of initial integration.
- Keep GH credentials out of the CRX bundle.
- Do not store Suno cookies or session credentials in SQLite or GitHub.
- Audio capture/export must remain explicit user actions.
- Do not blindly retry a prompt submission after a timeout if Suno may already have accepted it.

## Related planning

See [bonsai/crxman#41](https://github.com/bonsai/crxman/issues/41) and the integration inventory issues #48–#51 in that repository. Follow the implementation checklist in [sunomono#7](https://github.com/bonsai/sunomono/issues/7).
