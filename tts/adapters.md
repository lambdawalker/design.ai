# Model adapter contract

[Back to index](README.md)

Concrete interface: [server adapter.py](https://github.com/lambdawalker/python.tts.api.server/blob/main/src/tts_api_server/adapter.py). See the [adapter authoring guide](https://github.com/lambdawalker/python.tts.api.server/blob/main/docs/adapters.md) and [fake adapter](https://github.com/lambdawalker/python.tts.api.server/blob/main/src/tts_api_server/fake.py). The fake produces silence and is not a model integration.

An adapter integrates one engine with shared application services. It does not host HTTP or MCP routes.

## Required responsibilities

1. List exact model profiles and their readiness.
2. Provide versioned capabilities, guidance, and extension schemas per profile and feature.
3. Load and release models in the engine's own environment.
4. Validate native constraints without rewriting caller content.
5. Resolve registered voice references into native inference arguments.
6. Execute supported synthesis, design, or conversion operations.
7. Emit actual lifecycle stages and measured progress when available.
8. Return audio plus format/sample-rate metadata and execution provenance.
9. Declare incremental audio and cancellation behavior honestly.

A profile identifies checkpoint, checkpoint revision, adapter version, and relevant inference mode. Separate profiles are required when capabilities differ.

## Conceptual service boundary

| Adapter operation | Responsibility |
| --- | --- |
| list_profiles | Describe models and readiness without unnecessary model loading |
| get_capabilities(profile) | Report effective feature combinations and limits |
| get_guidance(profile, feature) | Return concise model-specific instructions |
| validate(profile, operation, request) | Check native constraints; return structured errors |
| load / unload | Manage inference resources |
| synthesize | Generate speech from caller-authored input |
| design_voice | Optional description-driven candidate generation |
| convert_voice | Optional source-audio transformation |

These names express responsibilities. The linked Python adapter interface is the concrete implementation contract; `execute` dispatches supported generation operations. Workers provide asset access, progress emission, and cancellation signals. The adapter must not create a competing queue or persistence layer.

## Native mapping versus rewriting

Mapping the API's `instructions` field to a model argument named `instruct` is allowed. Replacing its contents, inserting expression tags, removing tags, translating language, or silently changing a requested voice is prohibited.

Technical audio decoding or required resampling must be declared and recorded; they must not become undisclosed denoising, trimming, or semantic input repair.

Known unsupported controls return errors. Native runtime errors are translated into the shared domain error format. Never report an unavailable feature as successful by substituting another operation.

## Initial integration notes

- Fish: adapt existing local synthesis and reference behavior. Native tags remain the caller's responsibility. Do not infer that every upstream S2 capability is exposed by the installed local serving path.
- Chatterbox: preserve distinctions among standard, multilingual, Turbo, and other installed variants. Reuse the intent of existing job persistence, but conform to the shared contract. Voice conversion is distinct from reference registration.
- Qwen3-TTS: define separate Base, CustomVoice, and VoiceDesign profiles, with size-specific capabilities. Do not advertise instruction control for every checkpoint. Reference transcript requirements depend on cloning mode.

## Concurrency and provenance

Declare tested concurrency; use serialized inference by default. Record checkpoint revision, adapter version, native parameters, seed when supported, and output encoding. A seed does not guarantee equivalent output across engines or hardware.

Cancellation is cooperative where native inference cannot be interrupted safely. A cancellation request is not proof the GPU operation has stopped.
