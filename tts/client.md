# Importable Python client

[Back to index](README.md)

Implementation: [python.tts.api.client](https://github.com/lambdawalker/python.tts.api.client). Start with its [README](https://github.com/lambdawalker/python.tts.api.client/blob/main/README.md) and [wire assumptions](https://github.com/lambdawalker/python.tts.api.client/blob/main/docs/contracts.md). The counterpart is [python.tts.api.server](https://github.com/lambdawalker/python.tts.api.server).

The client is a lightweight HTTP consumer for Python applications, scripts, and agent runtimes. It does not load Torch, CUDA, model weights, or host MCP.

## Public surface

Provide synchronous and asynchronous clients with equivalent behavior. Proposed methods:

| Method | HTTP operation |
| --- | --- |
| list_models | GET /v1/models |
| capabilities | GET /v1/capabilities |
| guidance | GET /v1/guidance/{feature} |
| upload_asset / download_asset | POST /v1/assets and GET /v1/assets/{id} |
| list_voices / register_voice / delete_voice | Voice resource endpoints |
| validate_speech | POST /v1/speech/validate |
| generate_speech | POST /v1/speech |
| design_voice | POST /v1/voice-designs |
| convert_voice | POST /v1/voice-conversions |
| get_job / events / cancel_job | Job lifecycle endpoints |

Generation returns a job handle promptly. A separate `wait()` convenience operation can block with a caller-defined deadline. Timeout or closing the client does not cancel the server job.

## Illustrative use

This illustrates the implemented SDK interface. Import `TTSClient` from `tts_api_client`; configure the token and a real adapter supporting the requested profile/voice before running it. See the linked client quickstart for a complete setup.

```python
with TTSClient(base_url="http://qwen-server:8000", api_key=token) as client:
    capabilities = client.capabilities(model="default")
    guidance = client.guidance(feature="tts", model="default")
    voices = client.list_voices(model="default")

    # The person or agent reads these results and authors suitable input.
    request = {
        "model": "default",
        "text": "You are already here!",
        "voice": {"alias": "narrator"},
        "instructions": "Sound pleasantly surprised.",
    }
    client.validate_speech(**request)
    job = client.generate_speech(**request, idempotency_key="request-123")
    completed = job.wait(timeout=600)
    client.download_asset(completed.results[0].asset_id, "speech.wav")
```

The example requires a profile that supports instructions. The client must not decide how to rewrite that request for a different engine.

## Client responsibilities

- Typed request/result objects and structured exceptions preserving server error codes, field paths, and retryability.
- Connection timeouts distinct from job wait deadlines.
- SSE reconnection with the last event ID and polling fallback.
- No automatic regeneration following transport failure. Retry submission only with the same idempotency key and unchanged payload.
- Stream asset uploads/downloads without embedding large audio in model context.
- Cache capabilities and guidance by deployment, resolved model, and revision/ETag. Changing base URL invalidates that context.
- Keep credentials scoped to the configured deployment; do not forward authorization to arbitrary result hosts.
- Preserve text and instructions exactly; reject no native tag merely because the SDK does not recognize it.

Changing the base URL preserves API mechanics. Callers still refresh guidance and resolve valid model/voice selections. The SDK cannot guarantee identity portability.

## MCP relationship

An AI host that supports MCP can connect directly to the server's MCP interface without importing this client. A future standalone MCP bridge may use this library to reach HTTP-only deployments, but is not part of the initial SDK.

## Anonymous session access

See [anonymous sessions](sessions.md) for token issuance, resource isolation, expiration,
revocation and client lifecycle. Shared `--no-auth` access remains a separate mode.
