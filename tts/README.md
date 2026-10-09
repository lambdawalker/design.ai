# Interchangeable local TTS services

Design baseline: 2026-10-09. Status: architecture agreed in discussion; detailed contracts proposed for implementation. No runtime implementation is included here.

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

Implementation repository names suggested during discussion are `python.apexfission.tts.server` and `python.apexfission.tts.client`. They are proposed names, not dependencies that already exist. This folder in design.ai is the central specification.

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

This design defines component boundaries and the initial contract. Executable OpenAPI/JSON Schemas, SDK implementation, packaging, authentication deployment profiles, and GPU validation are implementation deliverables. Future additions include a standalone MCP-to-HTTP bridge and duplex conversational audio sessions; neither is required for the first implementation.
