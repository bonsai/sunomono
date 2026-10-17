# Data contract — initial proposal

JSON Schema and SQL migrations should be added together in a follow-up change.

## Conventions
- IDs: UUID strings independent of titles, DOM selectors, and filenames.
- Timestamps: ISO 8601 UTC strings.
- Schema version: explicit integer in exported documents.
- File paths: Workspace-relative paths with forward slashes; reject absolute paths and traversal segments.
- Audio bytes: files, not SQLite BLOBs.
- Unknown values: null or documented status; never invent a successful state.

## Entities

### Project
id, name, created_at, updated_at

### Album
id, project_id, title, status, created_at, updated_at

### Track
id, album_id (nullable), title, prompt_ref (nullable), source_url (nullable), generation_status, created_at, updated_at

### Asset
id, track_id (nullable), kind, relative_path, mime_type, size_bytes, sha256, created_at

### GenerationRun
id, track_id, status, started_at, completed_at (nullable), error_code (nullable), retry_count

### Event
id, entity_type, entity_id, event_type, payload_json, occurred_at

## Initial status vocabulary
- generation_status: planned, queued, running, succeeded, failed, unknown
- asset kind: audio_master, audio_preview, image, prompt, other
- run status: queued, running, succeeded, failed, cancelled

## Export envelope
schema_version, exported_at, application_version, projects[], albums[], tracks[], assets[], generation_runs[], events[].

Exports must not include secrets, session tokens, or authentication cookies.

## Integrity rules
- Foreign keys must reference existing rows.
- A relative_path must remain inside the selected Workspace.
- SHA-256 is optional until a file has been inspected; if present, it must be a 64-character lowercase hexadecimal digest.
- File paths are references, not proof that a file exists.
- Event payloads must be JSON and must not contain credentials.
- Migrations are forward-only by default; back up before destructive transformations.
