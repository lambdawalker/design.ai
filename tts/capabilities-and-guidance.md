# Capabilities and guidance

[Back to index](README.md)

Capabilities answer what can be requested. Guidance explains how a caller should construct good inputs. The caller, whether an agent or person, owns text, native tags, and instructions.

## Capability scope

Capabilities must identify the exact model profile, model revision, adapter version, capability revision, and readiness (ready, loadable, disabled, or unavailable). Report only behavior exposed by the installed adapter.

Include:

- Features: TTS, voice cloning/reference conditioning, voice design, voice conversion.
- Supported languages, explicit auto-detection behavior, voice kinds and defaults.
- Separate instruction support and native inline control syntax.
- Scope of controls: utterance, span, event, or speaker/turn.
- Supported feature combinations, not merely independent booleans.
- Reference requirements: count, audio formats, duration limits, transcript requirements by mode.
- Input limits with units; distinguish hard limits from recommendations.
- Output formats/sample rates and whether resampling is performed.
- Native incremental audio separately from job lifecycle notifications.
- Cancellation and progress granularity.
- Namespaced extension schemas and guidance links.

An upstream model supporting streaming does not mean the installed adapter exposes streaming. Instructions on one checkpoint and cloning on another do not imply instruction-controlled cloning is available.

## Illustrative capability fragment

```json
{
  "api_version": "1.0",
  "capabilities_revision": "17",
  "model": "qwen3-tts-1.7b-customvoice",
  "availability": "ready",
  "features": {
    "tts": {"supported": true},
    "voice_cloning": {"supported": false},
    "voice_design": {"supported": false}
  },
  "controls": {
    "instructions": {"supported": true, "scope": "utterance"},
    "inline_tags": {"supported": false}
  },
  "guidance": {
    "tts": "/v1/guidance/tts?model=qwen3-tts-1.7b-customvoice"
  }
}
```

This is a fragment, not a complete runtime claim. The implementation adds actual limits, versions, voices, and readiness.

## Guidance document

Each feature document contains model/profile identity, revision, concise summary, required inputs, input rules, examples, recommended practices, known limitations, and authoritative source links.

```json
{
  "feature": "tts",
  "model": "qwen3-tts-1.7b-customvoice",
  "revision": "3",
  "summary": "Synthesize with a preset voice and separate delivery instructions.",
  "input_rules": [
    "Put spoken words in text.",
    "Put delivery directions in instructions.",
    "Select a preset voice returned by list_voices.",
    "Do not use inline expression tags."
  ],
  "example": {
    "text": "You are already here!",
    "instructions": "Sound pleasantly surprised."
  },
  "limitations": [
    "Instructions guide delivery but do not guarantee exact acoustic behavior."
  ]
}
```

Examples are fragments when required fields are omitted for brevity; production guidance must identify that explicitly or provide complete valid requests.

## Feature-specific content

- TTS: native syntax, instruction placement, speaker/turn syntax, pronunciation controls, language selection, text limits.
- Cloning: reference quality, recommended length, hard bounds, transcripts, mode differences, identity limitations.
- Voice design: description structure, preview text, output candidates, explicit registration for reuse.
- Conversion: source audio versus target reference, supported formats and duration constraints.

For native tags, document exact syntax, scope, placement, aliases, and examples. Mark catalogs as exhaustive or non-exhaustive. Natural-language controls cannot always be captured in an enum.

Do not advertise automatic stripping, translation, guessed emotions, or silently ignored fields.

## Versioning and consistency

Use ETags for caching. Tie guidance to the model profile and adapter revision. A supplied stale `guidance_revision` causes a pre-submission error with the current revision and guidance link. Omitting it does not trigger mandatory read-tracking.

Maintain HTTP schemas, MCP schemas, validation rules, and complete examples from shared definitions where possible. Guidance prose supplements those schemas rather than overriding them.
