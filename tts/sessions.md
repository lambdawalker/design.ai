# Anonymous sessions

[Back to index](README.md)

Anonymous sessions allow public admission while isolating resources by an opaque bearer
credential. Sessions belong to a client identity, not a TCP connection, IP address or MCP
transport session. Reconnecting or restarting the server does not change ownership.

## Deployment modes

- Configured tokens: `TTS_API_TOKENS` maps credentials to persistent caller IDs.
- Anonymous sessions: `--anonymous-sessions` permits anyone to create an isolated session.
- Shared anonymous access: `--no-auth` makes all callers share the existing `local` identity.

The two CLI flags are mutually exclusive. Anonymous-session mode rejects configured
static tokens to avoid ambiguous credential policy. Existing modes retain their behavior.

## Wire contract

`POST /v1/sessions` accepts an empty request without authentication only in anonymous-session
mode. It returns HTTP 201 and `Cache-Control: no-store`:

```json
{
  "session_id": "session_opaque-id",
  "access_token": "opaque-secret",
  "token_type": "Bearer",
  "expires_at": "2026-10-11T08:00:00+00:00"
}
```

All other endpoints require `Authorization: Bearer <access_token>`, including discovery,
HTTP jobs/events/assets/voices and MCP. The server generates at least 256 bits of token
entropy and stores only a cryptographic token hash. Session IDs are identifiers, not
credentials. Never expose tokens through job provenance, URLs, logs or listing endpoints.

`DELETE /v1/sessions/current` requires the session token and returns 204 with no-store.
It revokes that credential. Missing, unknown, expired or revoked credentials return 401
`authentication_required`. Other sessions' resource IDs return the existing not-found
errors. There is no session listing or resource transfer endpoint.

Sessions persist in the server's SQLite state with an absolute expiration (default 86,400
seconds, configurable with `--session-ttl`). `--max-sessions` defaults to 10,000 active
sessions; admission returns 429 `session_limit_reached` when full. Expired rows are cleaned
before admission. This is a storage bound, not a per-IP abuse/rate-limiting system.

Expiration/revocation denies new requests. Already accepted work can finish and does not
automatically cancel jobs. HTTP job SSE stops when its session expires or is revoked;
reconnection then returns 401. Already started downloads or MCP responses may finish.
Resource retention is independent of session expiration. A new session cannot recover an
expired or lost session's resources. Static-token and shared-local resources stay separate.

## Python client

Both sync and async clients accept `anonymous_session=True` for lazy creation before the
first operation. Concurrent first operations on the same client create just one session.
`session_token="..."` resumes a saved credential; it is exclusive with `api_key`.
`create_session()` explicitly creates and activates a fresh session, `session_token` exposes
the credential for caller-managed secure storage, and `revoke_session()` revokes the current
credential. Closing a client does not revoke it. A supplied session token does not require
session creation. Never automatically replace a credential on 401 or retry submissions.
Session creation is not retried automatically after an ambiguous network failure.

MCP clients bootstrap with the HTTP session endpoint, then attach the same bearer token to
`/mcp`. MCP transport session IDs are not authentication credentials.

## Implementation and validation

- [Shared server](https://github.com/lambdawalker/python.tts.api.server): persisted credentials,
  CLI modes, middleware and session endpoints.
- [Python client](https://github.com/lambdawalker/python.tts.api.client): matching sync/async
  lifecycle and automatic creation.
- [Qwen deployment](https://github.com/lambdawalker/dgxspark.qwen3TTS): dependency pins and launch guide.

Acceptance covers cross-session status/events/cancellation/download/voice denial, caller-scoped
idempotency, restart persistence, expiration, revocation, admission bounds, HTTP/MCP parity,
client reconnection/resumption and unchanged existing authentication modes.
