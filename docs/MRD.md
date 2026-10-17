# MRD — sunomono

## Product statement
sunomono is a local-first Suno production workspace combining a Manifest V3 extension, browser-local SQLite WASM/OPFS, and a separately selected local folder for audio and production artifacts.

## Problem
Suno helpers, prompt definitions, KEEP state, recording/conversion tools, and track metadata can become fragmented across extensions, repositories, and local folders. Stable IDs and clear data boundaries are needed before codebases are merged.

## Users and jobs
- Creator manages prompts, clips, tracks, albums, and selected audio artifacts.
- Creator tracks which clips are kept, processed, converted, or assigned to an album.
- Local agent/CLI validates metadata, organizes files, and maintains specifications.
- Maintainer backs up and restores metadata without relying on an open Suno tab.

## Principles
1. Track/clip IDs are stable and independent of titles or transient DOM selectors.
2. Browser DB and local artifact storage are distinct systems.
3. GitHub specifications and schemas are versioned and reviewable.
4. Recording, export, overwrite, and deletion are explicit user actions.
5. Existing repositories are inventoried before migration; integration is incremental and reversible.
6. Capability failures are visible; persistence is never silently faked.

## MVP scope
- MV3 extension skeleton and capability status.
- SQLite WASM in a Worker, using OPFS after compatibility checks.
- Initial schema for albums, tracks, assets, generation runs, and events.
- Track metadata CRUD and search.
- User-selected Workspace folder for file inspection/export where supported.
- JSON metadata export/import and integrity checks.
- Tests for persistence, migrations, validation, and recovery.

## Non-goals
Multi-device sync, fully autonomous generation, storing audio blobs inside SQLite, silent recording, private API use/DRM circumvention, immediate replacement of existing repositories, mandatory always-on localhost bridge.

## Success measures
- Record survives extension restart in tested environment.
- Metadata export imports into a clean database and validates.
- Asset references can be checked for missing files and hash mismatches.
- User can see whether DB and Workspace are connected and healthy.
- Existing tools remain recoverable during migration.

## Risks
OPFS availability/locking vary by browser context and VFS; extension Worker/WASM loading may need packaging adjustments; browser origin storage needs explicit backup; local folder access requires user selection; legacy extensions may collide on service workers, commands, permissions, and storage keys.
