# Research and conformance

[Back to index](README.md)

Implementation tests: [shared server suite](https://github.com/lambdawalker/python.tts.api.server/tree/main/tests) and [client suite](https://github.com/lambdawalker/python.tts.api.client/tree/main/tests). See the server [validation boundaries](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/testing.md). These exercise the fake adapter and protocol behavior; actual engine/GPU validation remains separate.

Research snapshot: 2026-10-09. These findings inform the design; they are not GPU benchmarks, guarantees of DGX Spark compatibility, or permanent capability manifests.

## Existing forks

| Fork | Inspected behavior | Integration consequence |
| --- | --- | --- |
| [Fish Speech](https://github.com/lambdawalker/dgxspark.fish-speech/blob/main/docs/en/local-api-client.md) | Local audio responses, reference management, streaming path | Wrap in shared services; preserve documented native inputs |
| [Chatterbox](https://github.com/lambdawalker/dgxspark.chatterbox/blob/master/docs/http-api.md) | Persisted jobs, SSE reconnection, reference voices, conversion | Converge on shared job and error contracts |
| [Qwen3-TTS](https://github.com/lambdawalker/dgxspark.qwen3TTS/blob/40c096b4b759b3b253a6f15922615ce4d09d7817/docs/api.md) | Shared HTTP/MCP adapter with five checkpoint profiles; [CPU conformance tests](https://github.com/lambdawalker/dgxspark.qwen3TTS/tree/40c096b4b759b3b253a6f15922615ce4d09d7817/tests/api) | Run the hardware smoke example on DGX Spark; GPU/acoustic results remain unverified |

Recheck these links and the actual adapter source when implementation begins; repositories evolve.

## Broader local model research

| Project | Design-relevant difference |
| --- | --- |
| [Fish Speech](https://github.com/fishaudio/fish-speech) | Native inline expression controls and multi-speaker workflows |
| [Chatterbox](https://github.com/resemble-ai/chatterbox) | Variant-specific controls, including paralinguistic tags |
| [Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS) | Base cloning, CustomVoice presets, and VoiceDesign are separate model profiles |
| [CosyVoice](https://github.com/QwenAudio/CosyVoice) | Separate instructions can coexist with inline controls |
| [IndexTTS](https://github.com/index-tts/index-tts) | Voice identity and emotion may use different reference audio or control inputs |
| [F5-TTS](https://github.com/SWivid/F5-TTS) | Reference audio/transcript requirements and composed multi-style workflows |
| [Kokoro](https://github.com/hexgrad/kokoro) | Preset-voice synthesis is useful without universal cloning or instruction support |
| [Parler-TTS](https://github.com/huggingface/parler-tts) | Descriptive conditioning separate from spoken text |

Local availability does not establish a shared license or commercial permission. Code and weight licenses must be checked separately when packaging a deployment.

## Shared contract acceptance checks

1. Equivalent HTTP and MCP requests call the same application service and yield equivalent results/errors.
2. Text, tags, punctuation, and instructions reach the adapter unchanged. No stripping, translation, trimming of tags, or automatic repair.
3. Unsupported explicit fields and invalid combinations produce actionable errors before queueing.
4. Non-exhaustive guidance catalogs do not become false closed-enum validators.
5. Capabilities reflect the exact installed checkpoint and adapter, including incompatible feature combinations.
6. Guidance revisions and all complete examples agree with validation schemas.
7. Changing base URL does not reuse cached model assumptions, local voice IDs, or credentials for another host.
8. Duplicate submissions with the same idempotency key do not duplicate inference; changed payloads conflict.
9. SSE reconnection and polling preserve job identity; disconnect never cancels a job.
10. Restart and cancellation outcomes follow the published lifecycle rules.
11. Expiration and authorization apply equally to jobs and assets through both interfaces.
12. The client imports and runs without Torch, CUDA, model weights, or MCP dependencies.

Use a fake adapter for deterministic contract tests. Add model-specific smoke tests on the actual Spark for inference, references, available controls, cancellation boundaries, and any claimed incremental audio. Only advertise features supported by those adapters; separate untested hardware status from API conformance.

## Delivery order

First define executable schemas and fixtures from this design. Then implement shared services and a fake adapter, HTTP and MCP interfaces, and the lightweight client. Integrate each engine independently and run the same conformance suite. Preserve or explicitly deprecate legacy endpoints during migration; this document does not authorize their silent removal.
