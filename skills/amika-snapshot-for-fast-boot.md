---
name: Capture a sandbox snapshot for fast boots
description: Configure a sandbox once, capture it as a reusable base-image snapshot, and launch new sandboxes from it so agents boot fast with dependencies pre-installed.
api: openapi/amika-openapi.json
base_url: https://app.amika.dev/api/v0beta1
operations:
- "POST /sandboxes"
- "GET /sandbox-snapshots/scrub-preview"
- "POST /sandbox-snapshots"
- "GET /sandbox-snapshots/{ref}"
- "GET /sandbox-snapshots"
---

# Capture a sandbox snapshot for fast boots

Snapshots turn a fully-configured sandbox into a reusable base image so new
sandboxes start with dependencies and setup already baked in.

## Auth
`Authorization: Bearer <token>` (API key or OAuth device-flow token) on every call.

## Steps
1. **Configure a source sandbox** — `POST /sandboxes`, then run your setup so the
   environment is in the desired state.
2. **Preview the scrub** — `GET /sandbox-snapshots/scrub-preview` to review what
   secrets/state will be stripped before the image is captured
   (`SandboxScrubPreviewResponse`).
3. **Capture the snapshot** — `POST /sandbox-snapshots` with the
   `source_sandbox_id` (`CreateSandboxSnapshotRequest`). This returns `202` and
   starts an async capture producing a `SandboxSnapshot`.
4. **Poll for readiness** — `GET /sandbox-snapshots/{ref}` until `state` is ready.
5. **Reuse it** — reference the snapshot as a repository's `default_snapshot` /
   `base_snapshot` so future `POST /sandboxes` boot from it. List available
   snapshots with `GET /sandbox-snapshots`.

## Conventions & errors
- Capture is asynchronous (`202 Accepted`); always poll the `{ref}` until ready.
- Errors use the custom `{ type, error_code, message, details }` envelope with an
  `x-trace-id` header; watch for `409` (snapshot conflict) and `502` (provider
  failure). See `errors/amika-problem-types.yml` and `conventions/amika-conventions.yml`.
