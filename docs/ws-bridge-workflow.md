# WS Bridge workflow: GitHub prompt to recorded track

## Goal
Use GitHub as the prompt/spec source, the GH API to retrieve it, a local agent to validate and dispatch work, a WebSocket bridge to communicate with the CRX, and Suno Web for user-visible generation. Record the resulting track through the existing recorder workflow and index it in SQLite WASM/OPFS.

## Sequence

    GitHub Prompt/Spec
          |
          v
    Local Agent -- GH API read
          |
          v
    Validate schema + create run_id
          |
          v
    WS Bridge -- versioned JSON-RPC
          |
          v
    CRX validates command, target tab, and approval
          |
          v
    Suno Web UI receives prompt
          |
          v
    Observe generation / identify clip
          |
          +---- uncertain completion --> needs_user_action
          |
          v
    User explicitly starts recording
          |
          v
    Recorder saves audio to Local Workspace
          |
          v
    SQLite OPFS indexes run, clip, asset, and events
          |
          v
    Export result manifest / optionally commit metadata to GitHub

## Component responsibilities

- **GitHub:** versioned prompt/spec and reviewed product contracts.
- **GH API client / Local Agent:** fetch a specified prompt, validate it, create a run ID, dispatch a request, correlate responses, and produce result manifests. Never embed a personal access token in the CRX bundle.
- **WS Bridge:** transport and event channel between local tools and the extension. Before implementation, inspect the actual bridge implementation and record its real endpoint, protocol, authentication, and approval model. Do not invent methods.
- **CRX:** validates the caller and request, confirms the correct Suno tab, fills the supported page UI, and reports observable events. It should not claim generation succeeded unless completion is verifiable.
- **Suno Web:** the UI through which the creator submits/monitors generation.
- **Recorder:** captures audio only after explicit user action, then returns the resulting file metadata/path.
- **SQLite WASM/OPFS:** stores production state, stable IDs, event history, and artifact references. It does not store audio bytes.
- **Local Workspace:** stores actual audio and other artifacts; use safe relative paths.

## State machine

- queued
- prompt_loaded
- submitted
- generating
- generated
- generated_unconfirmed
- needs_user_action
- recording_requested
- recorded
- indexed
- failed
- cancelled

generated means the extension has positive evidence of completion. If only a page transition or timer was observed, use generated_unconfirmed and ask the user to confirm.

## Correlation contract

Each operation should carry:
- protocol_version
- request_id: unique per transport request
- run_id: stable ID for one generation attempt
- idempotency_key: prevents accidental duplicate submission after retry
- prompt_ref: stable GitHub file/path plus revision or content hash
- track_id: stable local track ID, assigned before or at submission
- clip_id: captured from the resulting song only when actually observed
- asset_id: stable ID for the recorded file
- occurred_at: ISO 8601 UTC
- status and structured error when applicable

Do not use title alone as an identifier. Do not invent clip IDs or infer a completed recording from a request acknowledgement.

## Transport envelope proposal

The following is a conceptual contract, not an assertion about the existing bridge method names:

    {
      "protocol_version": 1,
      "request_id": "uuid",
      "run_id": "uuid",
      "idempotency_key": "stable-key",
      "method": "method-to-be-confirmed-after-bridge-inventory",
      "params": {}
    }

Responses and events must echo correlation IDs. Define allowed methods and payload schemas only after the existing WS Bridge implementation is inspected.

## Reliability

- Use timeouts and explicit error states.
- A transport retry must not automatically submit a second generation if the first request may have reached Suno.
- Keep generation submission and recording as separate actions and permissions.
- On lost connection, query/observe current tab state before retrying.
- If the clip cannot be identified, stop automatic association and request user confirmation.
- If recording or file persistence fails, preserve the run metadata and surface recovery instructions.
- Keep sensitive content, cookies, and credentials out of logs and manifests.

## Security

- Bind a local bridge to loopback only; do not expose an unauthenticated listener to the LAN.
- Validate sender origin, extension identity, command allowlist, payload schema, and target tab.
- Never forward arbitrary JavaScript or arbitrary URLs through a generic bridge command.
- Do not store Suno session cookies or credentials in SQLite or GitHub.
- Recording must be user-initiated and visible.

## Implementation order

1. Inventory the actual WS Bridge and Recorder protocol and permissions.
2. Freeze the versioned message/data contracts.
3. Implement read-only health/status and a test event round trip.
4. Implement prompt retrieval and validation without submitting to Suno.
5. Add one-run prompt dispatch with confirmation and correlation IDs.
6. Add clip identification and explicit recording workflow.
7. Add metadata export and recovery tests.