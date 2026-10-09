# Jobs, errors, and compatibility

[Back to index](README.md)

Implementation: [shared service](https://github.com/lambdawalker/python.tts.api.server/blob/main/src/tts_api_server/service.py), [SQLite store](https://github.com/lambdawalker/python.tts.api.server/blob/main/src/tts_api_server/store.py), and [operational limits/retention](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/deployment.md).

## Job lifecycle

Generation submissions return HTTP 202 or the equivalent MCP structured job result.

```json
{
  "id": "job_123",
  "status": "queued",
  "operation": "tts",
  "model": "resolved-profile",
  "status_url": "/v1/jobs/job_123",
  "events_url": "/v1/jobs/job_123/events",
  "results": []
}
```

States: queued, running, succeeded, failed, cancelled. Stages such as loading_model, generating, and encoding are separate from status. Cancellation uses a separate `cancellation_requested` flag until confirmed.

Progress percentages are nullable. Emit percentages only when measured meaningfully; lifecycle stages must not masquerade as neural inference progress. A successful job returns result asset IDs, media metadata, and model/adapter provenance.

## Persistence and reconnection

Persist job metadata, lifecycle events, and results. Disconnecting HTTP, SSE, or MCP does not cancel inference. SSE uses monotonic per-job event IDs and accepts Last-Event-ID; polling provides a fallback. Heartbeats maintain liveness but are not progress.

Initial restart policy: preserve completed results and mark interrupted queued/running jobs failed with `server_restarted`. Do not automatically rerun inference. This conservative policy may later evolve through an explicit versioned operational change.

Advertise event and result retention and expiration timestamps. Expired results return 410 `asset_expired`. If an SSE cursor predates retained events, return 409 `event_history_expired` and a status URL.

## Idempotency

Generation submissions accept Idempotency-Key; MCP submission tools accept an equivalent `idempotency_key` argument. Scope keys by caller and operation.

The same key and same validated request return the original job. Reusing a key for a different request returns 409 `idempotency_conflict`. Persist the request fingerprint and resolved model with the key; retries must not resolve a changed default into a new job. Publish key retention.

The client must not retry generation with a new key merely because a connection failed. A deliberate new attempt after a failed job uses a new key.

## Errors

```json
{
  "error": {
    "code": "unsupported_parameter",
    "message": "This model profile does not support separate instructions.",
    "field": "instructions",
    "retryable": false,
    "guidance_url": "/v1/guidance/tts?model=selected-profile"
  }
}
```

| HTTP status | Representative codes |
| --- | --- |
| 400 | malformed_request |
| 401 / 403 | authentication_required / forbidden |
| 404 | model_not_found, voice_unavailable, asset_not_found, job_not_found |
| 409 | guidance_revision_mismatch, idempotency_conflict, event_history_expired |
| 410 | asset_expired |
| 413 | upload_too_large |
| 422 | unsupported_feature, unsupported_parameter, unsupported_language, invalid_reference, invalid_request |
| 429 | queue_full |
| 503 | model_unavailable |

Known unsupported operations use 422 across deployments. Unknown routes use ordinary 404. Errors during accepted execution appear on the job as failed; a successful status lookup still returns HTTP 200.

MCP tool execution errors preserve the same domain code and details using the MCP tool-error mechanism; protocol-level errors remain distinct.

## Cancellation and cleanup

Queued jobs can be cancelled before execution. Running cancellation is cooperative and may wait for a safe boundary. If execution already completed, return its terminal state. Do not claim cancellation while inference is still running.

Asset deletion/retention must not remove references needed by active jobs. Removing a voice registration prevents future use but does not mutate an already accepted job's reference snapshot.

## Version compatibility

Use /v1 for the initial major HTTP contract. Clients reject unsupported major versions. Additive response fields may be ignored by older clients; unknown request fields must be rejected by servers. Breaking field meanings or operation semantics require a new major contract.

Audio streaming is optional and distinct from SSE. Do not label complete buffered output as native incremental generation. Duplex sessions and their transport are future scope.
