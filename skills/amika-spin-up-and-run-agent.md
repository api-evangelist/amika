---
name: Spin up a sandbox and run a coding agent
description: Create an Amika sandbox pre-loaded with a repository, run a coding agent (Claude/Codex) inside it, and read the result — the core Amika software-factory flow.
api: openapi/amika-openapi.json
base_url: https://app.amika.dev/api/v0beta1
operations:
- "POST /sandboxes"
- "GET /sandboxes/{id}"
- "POST /sandboxes/{id}/start"
- "POST /sandboxes/{id}/agent-send"
- "GET /sandboxes/{id}/sessions/latest"
- "POST /sandboxes/{id}/stop"
- "DELETE /sandboxes/{id}"
---

# Spin up a sandbox and run a coding agent

Use the hosted Amika API (`https://app.amika.dev/api/v0beta1`) to create an isolated
sandbox from a repository, run a coding agent in it, and collect the result.

## Auth
Send `Authorization: Bearer <token>` on every request, where the token is an Amika
API key (`AMIKA_API_KEY`) or an OAuth device-flow access token. Every response
carries an `x-trace-id` header — log it for support correlation.

## Steps
1. **Create the sandbox** — `POST /sandboxes` with the repository and agent
   configuration (`CreateSandboxRequest`). Capture the returned `Sandbox.id`.
2. **Ensure it is running** — poll `GET /sandboxes/{id}` until state is running;
   if stopped, call `POST /sandboxes/{id}/start`.
3. **Send work to the agent** — `POST /sandboxes/{id}/agent-send` with the
   instruction (`AgentSendRequest`); read the `AgentSendResponse`. For long jobs
   use `POST /sandboxes/{id}/agent-send-jobs` and poll
   `GET /sandboxes/{id}/agent-send-jobs/{job_id}`.
4. **Read the session** — `GET /sandboxes/{id}/sessions/latest` to inspect the
   `AgentSession` (status, transcript metadata).
5. **Tear down** — `POST /sandboxes/{id}/stop` to pause (preserving state) or
   `DELETE /sandboxes/{id}` to remove it.

## Conventions & errors
- List endpoints paginate by `cursor` + `limit` (see `conventions/amika-conventions.yml`).
- Errors return `{ type: "error", error_code, message, details }` (not RFC 9457).
  Handle `401/403` (auth/permission), `404` (missing sandbox), `409` (bad lifecycle
  transition), `502` (upstream sandbox-provider failure). See `errors/amika-problem-types.yml`.
- The API is beta (`v0beta1`); expect breaking changes.
