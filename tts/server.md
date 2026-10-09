# Shared server and HTTP API

[Back to index](README.md)

Implementation: [python.tts.api.server](https://github.com/lambdawalker/python.tts.api.server), with [OpenAPI/request schemas](https://github.com/lambdawalker/python.tts.api.server/tree/main/schemas), [wire contract](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/contracts.md) and [deployment profile](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/deployment.md).

The shared server is the application boundary. Adapters supply inference behavior; HTTP and MCP expose the same services. This v1 contract is implemented by the shared server; consult its executable schemas for exact wire fields.

## Endpoint contract

| Method and route | Behavior |
| --- | --- |
| GET /v1/models | Available model profiles, defaults, readiness, capability links |
| GET /v1/capabilities?model={id} | Effective capabilities; omitted model resolves the default |
| GET /v1/guidance/{feature}?model={id} | Versioned feature guidance for the selected profile |
| GET /openapi.json | Generated HTTP schema |
| POST /v1/assets | Multipart audio upload; returns asset metadata |
| GET /v1/assets/{id} | Download an authorized asset |
| GET /v1/voices?model={id} | Compatible voices, aliases, and provenance |
| POST /v1/voices | Register reference assets as a reusable voice |
| DELETE /v1/voices/{id} | Remove a registered voice; does not implicitly delete shared assets |
| POST /v1/speech/validate | Validate without inference or input modification |
| POST /v1/speech | Submit synthesis; returns 202 and a job |
| POST /v1/voice-designs | Submit description-based voice candidate generation |
| POST /v1/voice-conversions | Submit source-audio conversion to a target voice |
| GET /v1/jobs/{id} | Status, stage, results, and structured error |
| GET /v1/jobs/{id}/events | Reconnectable SSE lifecycle notifications |
| POST /v1/jobs/{id}/cancel | Request cancellation |

All deployments expose these application routes. Unsupported features return `unsupported_feature`; clients discover support before submitting work.

## Speech request

```json
{
  "model": "default",
  "text": "You are already here!",
  "voice": {"alias": "narrator"},
  "language": "en",
  "instructions": "Sound pleasantly surprised.",
  "output": {"format": "wav"},
  "guidance_revision": "3",
  "extensions": {}
}
```

This example is suitable only for a profile supporting separate instructions. A tag-capable model's caller writes native tags directly into `text` according to its guidance. No `portable-v1` format or compatibility rewriting policy exists.

- `text` is required and passed unchanged to the model API. Intrinsic model tokenization is not an application rewriting feature; document unavoidable native preprocessing.
- `model` defaults to the deployment's configured profile.
- `voice` accepts exactly one of `id` or `alias`. If omitted, the configured default is used only when valid for that profile; otherwise validation fails.
- `language` is optional only where model defaults or auto detection are documented. Never silently substitute languages.
- `instructions` is optional; supplying it to an unsupported profile is a validation error.
- `output` defaults to WAV. Advertise available encodings and sample rates.
- `guidance_revision` is optional. When supplied and stale, return `guidance_revision_mismatch` before queueing.
- `extensions` contains namespaced native controls with profile-specific schemas. Reject unknown namespaces or fields.
- Unknown top-level fields are rejected rather than ignored.

## Validation

Return `{"valid": true, "model": "...", "guidance_revision": "..."}` for valid input, or the standard validation error. Validation does not synthesize, rewrite, trim tags, or fix malformed native text. It checks structural types, required references, documented limits, compatible voices, and known constraints.

Arbitrary natural-language tags cannot always be validated semantically. Do not claim invalid syntax merely because a tag is absent from a non-exhaustive guidance example list.

## Reference assets and voice registration

Upload bytes using multipart HTTP. The server assigns an opaque asset ID and reports MIME type, sample rate, channels, duration, and size. Do not accept arbitrary server filesystem paths as public inputs.

A reference registration request uses `model`, an optional `alias`, and `references`: each reference contains `asset_id` and, when required, `transcript`. The transcript must match the recording. No hidden ASR transcription or reference editing occurs.

Registration stores reusable reference metadata and returns 201 with a voice ID. Expensive embedding preparation can happen inside the first synthesis job. Cache keys include the model revision.

## Optional generation workflows

Voice design accepts `model`, `description`, `preview_text`, optional supported `language`, `output`, and namespaced `extensions`. It returns a job whose results are candidate audio assets. Promoting a candidate to a reusable voice is an explicit reference registration operation.

Voice conversion accepts `model`, `source_asset_id`, target `voice`, `output`, and supported `extensions`. Voice conversion changes source audio; voice cloning conditions synthesis of new text. They are separate capabilities.

## Access and storage

Apply identical authorization to HTTP, MCP, jobs, voices, and assets. Credentials belong in connection configuration, not model tool arguments or logs. Enforce upload and queue limits. Asset downloads must respect access control and expiration. The concrete authentication profile is selected during deployment implementation.
