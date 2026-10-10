# MCP interface

[Back to index](README.md)

Implementation: [server MCP interface](https://github.com/lambdawalker/python.tts.api.server/blob/main/src/tts_api_server/mcp.py). Enable the optional `mcp` extra and `--mcp`; see [deployment/authentication](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/deployment.md).

## Placement

MCP is an optional interface of the shared server package, mounted at `/mcp` when enabled in the shared server deployment. It calls the same application services as HTTP.

Individual model adapters provide capabilities, guidance, and inference. They do not duplicate MCP tools. The Python client uses HTTP and has no required MCP dependency.

## Tools and application operations

| Tool | Shared operation |
| --- | --- |
| list_models | List model profiles |
| get_capabilities | Read effective capabilities |
| get_guidance | Read feature guidance |
| list_voices | List compatible voices |
| clone_voice | Register a reusable voice from existing reference asset IDs |
| delete_voice | Remove a registered voice |
| validate_speech | Validate a synthesis request without inference or rewriting |
| generate_speech | Submit a synthesis job |
| design_voice | Submit a voice-design job |
| convert_voice | Submit a voice-conversion job |
| get_job | Read status and result asset links |
| cancel_job | Request cancellation |

`clone_voice` maps to HTTP POST /v1/voices: it registers conditioning references; it does not fine-tune weights or generate new speech. Its tool description must make this explicit.

Schemas, validation, error codes, and behavior derive from the same definitions used by HTTP. Unsupported operations return `unsupported_feature`. Tool names stay stable across deployments; capabilities indicate availability.

## Agent workflow

1. Discover models and select a profile.
2. Read its capabilities and feature guidance.
3. Resolve a voice and required assets.
4. Author native text, tags, instructions, and controls.
5. Optionally validate; submit generation.
6. Track the independent job and retrieve resulting audio.

Tool descriptions tell agents to refresh guidance when deployment, model, or revision changes. Guidance retrieval is not evidence of understanding; validation remains necessary. Do not force an LLM call inside the adapter or server.

## Guidance and resources

Expose guidance through `get_guidance`, so tool-oriented clients can explicitly retrieve it. Optionally expose equivalent MCP resources such as `tts://guidance/{model}/{feature}`; both return the same revisioned content.

Optional prompt templates can explain workflows, but cannot be required for correctness. Essential field constraints belong in tool schemas and concise descriptions. Fetch lengthy best practices on demand.

## Audio and jobs

Generation tools return a job ID and status promptly. Job completion does not depend on keeping an MCP call or session alive. `get_job` returns metadata and authorized audio links; HTTP provides uploads, downloads, and SSE.

References must be uploaded through HTTP/client integration before submitting asset IDs. A remote MCP server cannot read a person's local filesystem merely because the agent supplies a local path. Do not imply automatic attachment upload support.

Do not place large base64 audio in routine tool results. Optional audio/resource result presentation depends on host capabilities and must not replace the durable asset.

## Access

Enforce the same authorization as HTTP. Keep credentials in host connection configuration, not tool arguments. Generation, cloning registration, deletion, and cancellation are mutating tools; capability and guidance retrieval are read-only.

Use the selected MCP SDK's supported protocol transport and authentication behavior during implementation. The `/mcp` route is a deployment choice, not a custom MCP protocol.

## Optional future bridge

A separate local MCP process could wrap the Python HTTP client for HTTP-only deployments. That is an alternative integration, not the primary placement and not a second implementation of application logic.

Reference: [MCP server concepts](https://modelcontextprotocol.io/specification/latest/server/index).

## Anonymous session access

See [anonymous sessions](sessions.md) for token issuance, resource isolation, expiration,
revocation and client lifecycle. Shared `--no-auth` access remains a separate mode.
