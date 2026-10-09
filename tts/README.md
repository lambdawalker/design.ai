# Interchangeable local TTS services

Design baseline: 2026-10-09. Status: central architecture and contract, with server and client implementations maintained in the repositories below. No runtime code is included in design.ai.

## Goal

Provide the same HTTP endpoints and MCP tools across local TTS deployments so a client can change its base URL, discover the selected model's capabilities and guidance, and submit suitable inputs.

People and AI agents are responsible for constructing model-appropriate inputs. The service must not strip tags, translate tags, rewrite instructions, infer missing instructions, or silently discard unsupported parameters. Validation does not guarantee acoustic quality or instruction adherence.

## Read in order

1. [Architecture and ownership](architecture.md)
2. [Shared server and HTTP API](server.md)
3. [Model adapter contract](adapters.md)
4. [Importable Python client](client.md)
5. [Server-side MCP interface](mcp.md)
6. [Capabilities and guidance](capabilities-and-guidance.md)
7. [Jobs, errors, and compatibility](jobs-and-errors.md)
8. [Research and conformance](research-and-conformance.md)

## Initial integrations

- [Fish Speech fork](https://github.com/lambdawalker/dgxspark.fish-speech)
- [Chatterbox fork](https://github.com/lambdawalker/dgxspark.chatterbox)
- [Qwen3-TTS fork](https://github.com/lambdawalker/dgxspark.qwen3TTS)

## Implementation sources

- [python.tts.api.server](https://github.com/lambdawalker/python.tts.api.server): shared HTTP/MCP services, adapter interface, persistent jobs/assets/voices, executable schemas and conformance tests.
- [python.tts.api.client](https://github.com/lambdawalker/python.tts.api.client): synchronous/asynchronous Python HTTP client, uploads/downloads, job handles and SSE recovery.
- [Qwen adapter](https://github.com/lambdawalker/dgxspark.qwen3TTS/blob/40c096b4b759b3b253a6f15922615ce4d09d7817/qwen_tts_api/adapter.py) and [DGX Spark deployment guide](https://github.com/lambdawalker/dgxspark.qwen3TTS/blob/40c096b4b759b3b253a6f15922615ce4d09d7817/docs/api.md): native Base, CustomVoice and VoiceDesign integration; [implementation PR](https://github.com/lambdawalker/dgxspark.qwen3TTS/pull/1).
- [Server wire schemas](https://github.com/lambdawalker/python.tts.api.server/tree/main/schemas), [adapter guide](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/adapters.md), and [deployment guide](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/deployment.md).

These names supersede the earlier proposed `python.apexfission.tts.*` names. This folder remains the central architectural specification. Concrete install commands, package versions and deployment limitations belong in the implementation repositories.

The initial shared server ships an explicit fake adapter that produces silent WAVs for contract testing. The Qwen integration above adds engine-local adapters and CPU conformance tests; DGX Spark inference and acoustic validation remain required. Fish and Chatterbox still need shared-contract adapters. A passing server/client test is not evidence of real-model inference.

## Agreed decisions

- One shared server package, integrated into each model environment through an adapter.
- HTTP and optional MCP interfaces share application services and validation.
- MCP belongs to the shared server, not to individual adapters or the Python client.
- The Python client uses HTTP and does not load model libraries.
- Callers discover capabilities and guidance and author the inputs themselves.
- No automatic tag stripping, translation, content repair, or silent fallback.
- Capabilities describe an exact model profile and its installed adapter.
- Jobs continue independently of a client's connection.
- Model-specific voice IDs, assets, languages, and controls are not automatically portable.

Earlier proposals for a portable tag language, automatic adaptation, and a rewriting prepare endpoint are superseded. The validation endpoint checks inputs without transforming them.

## Scope

This design defines component boundaries and the initial contract. Executable OpenAPI/JSON Schemas and server packaging are maintained in the server repository; the SDK is maintained in the client repository. The initial server uses a static bearer-token deployment profile, one inference worker and SQLite persistence. GPU validation remains an engine-integration deliverable. Future additions include a standalone MCP-to-HTTP bridge and duplex conversational audio sessions; neither is required for the first implementation.
