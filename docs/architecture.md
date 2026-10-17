# Architecture and proposed repository layout

    sunomono/
    ├─ README.md
    ├─ manifest.json
    ├─ package.json
    ├─ docs/
    │  ├─ MRD.md
    │  ├─ MVP.md
    │  ├─ architecture.md
    │  ├─ data-contract.md
    │  └─ adr/
    │     └─ 0001-storage-boundaries.md
    ├─ src/
    │  ├─ background/
    │  ├─ content/
    │  ├─ popup/
    │  ├─ sidepanel/
    │  ├─ db/
    │  │  ├─ worker/
    │  │  ├─ migrations/
    │  │  └─ repositories/
    │  ├─ workspace/
    │  ├─ contracts/
    │  └─ shared/
    ├─ schemas/
    ├─ tests/
    │  ├─ unit/
    │  ├─ integration/
    │  └─ fixtures/
    └─ scripts/

## Runtime responsibilities
- Popup/side panel: status and explicit user actions.
- Content scripts: minimal supported Suno page interaction; no persistent data ownership.
- Background service worker: extension lifecycle and message routing, not long-running DB ownership.
- DB Worker: SQLite WASM lifecycle, migrations, transactions, query API.
- Workspace adapter: user-selected directory, permission checks, safe relative paths, asset integrity.
- Contracts: shared types and runtime validation.

## Proposed boundaries
- UI must not issue arbitrary SQL.
- DB access goes through repositories/typed commands.
- Browser messages are versioned and validated.
- Workspace operations are separate from OPFS operations.
- Prefix extension-specific storage keys and commands with sunomono.
- Migrate existing extension features only after permissions and collision inventory.
