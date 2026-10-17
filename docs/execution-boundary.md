# Execution boundary: GitHub Actions and local runtime

## Principle
Run deterministic, repository-contained checks on GitHub Actions. Run browser-session, local-filesystem, audio-capture, and environment-specific work locally. Do not expose a personal machine as a public service merely to let a hosted runner reach it.

## Responsibility matrix

| Task | GitHub Actions | Local Agent / CRX |
|---|---|---|
| Prompt/spec schema validation | Primary | Validate again before dispatch |
| Markdown/ADR/manifest lint | Primary | — |
| Unit tests, build, package | Primary | Optional local reproduction |
| SQL migration syntax and fixture tests | Primary | — |
| Real MV3 OPFS persistence | Not assumed | Browser PoC |
| Fetch approved prompt by GitHub reference | May validate reference | Fetch/resolve for a run |
| Send prompt to an open Suno Web tab | No | CRX through WS Bridge |
| Observe generated song / resolve clip ID | No | CRX/user confirmation |
| Record audio from browser tab | No | User-triggered Recorder |
| Audio conversion, hashing, file inspection | Only for uploaded test artifacts | Local CLI/Workspace |
| Metadata result manifest validation | Primary | Generate then validate |
| Publish result metadata / issue / PR | With scoped token | Local Agent may submit |
| Store production audio | No default | Local Workspace |

## Job protocol
- Job request is a versioned JSON document tracked in GitHub.
- Required identifiers: job_id, run_id, prompt_ref, prompt_revision or content digest, created_at, requested_action.
- Local Agent claims/picks up work and records state durably before performing side effects.
- Use explicit states: queued, claimed, prompt_loaded, awaiting_user, submitted, generating, generated_unconfirmed, generated, recording, recorded, indexed, failed, cancelled.
- A timeout after submission is ambiguous. Reconcile current Suno tab and recent events before retrying; never blindly resubmit.
- Result manifest carries run_id, prompt revision/digest, status, timestamps, observed clip_id if available, and asset metadata. It does not include cookies, tokens, or raw audio by default.

## Recommended initial topology
1. GitHub Actions validates prompt files, schemas, migrations, tests, and package artifacts on push/PR.
2. Local Agent polls or explicitly pulls queued jobs from GitHub; local PC does not accept unsolicited inbound Internet connections.
3. Local Agent uses the existing WS Bridge to address the CRX. Confirm the bridge's actual protocol and security before implementing.
4. CRX submits to the Suno UI and reports only observable states.
5. User starts recording explicitly; Recorder writes to the selected local Workspace.
6. SQLite WASM/OPFS stores metadata and state; Local Agent produces a result manifest and submits metadata to GitHub when requested.

## Security
- Use least-privilege GitHub tokens; prefer GitHub App or fine-grained token scoped to the necessary repository/actions.
- Do not put GitHub credentials in the CRX bundle, prompt files, issue text, or logs.
- Bind local WebSocket bridge to loopback, authenticate callers, validate origins/senders, and allowlist methods.
- Do not store session cookies or credentials.
- Keep large audio files outside Git unless intentionally versioned; publish checksums and relative artifact references instead.
- GitHub Actions must not claim to have verified browser OPFS, local file permissions, or recording unless those tests actually ran on a suitable environment.

## CI workflow proposal
- pull_request and push to main: lint docs, validate JSON/JSON Schema, test migrations/fixtures, unit tests, build CRX.
- Package release artifact and upload as workflow artifact.
- Manual workflow_dispatch: optional metadata validation or release packaging.
- Do not automatically submit Suno prompts or start recording from CI.

## Failure behavior
- Bridge offline: leave job queued or paused and show a reconnect action.
- Local Agent offline: job remains queued; no hosted runner connects inward.
- Suno completion unclear: generated_unconfirmed and request user confirmation.
- Recorder save failed: retain run metadata, report error, permit explicit retry after checking whether the audio file exists.
- OPFS unavailable: display a clear unsupported/error state; do not silently use ephemeral memory and report success.