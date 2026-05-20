# 05 — Voice Pipeline: STT, Manager Persona, TTS

## Overview

Ex8 adds a voice interface. The architecture is:

```text
microphone → speech-to-text → manager persona → text-to-speech → speakers
```

Voice does not replace the agent. It is a transport layer around the same
booking conversation.

## Text Mode

`make ex8-text` uses stdin/stdout:

- you type a message,
- the manager persona replies,
- trace events are written.

This is free and useful for debugging.

## Voice Mode

`make ex8-voice` adds:

- Speechmatics for speech-to-text,
- Rime.ai for text-to-speech,
- local microphone and speaker access.

It needs:

- `SPEECHMATICS_KEY`,
- `RIME_API_KEY`,
- voice dependencies,
- microphone permissions.

## Manager Persona

The manager persona is an LLM wrapper with a strict system prompt. It plays
Alasdair, the pub manager.

It must enforce:

- party size at most 8,
- deposit at most £300,
- no invented policy rules,
- short in-character replies.

In our local version, `ManagerPersona.from_env()` uses the same provider config
as the rest of the agent. This prevents hidden Nebius hardcoding when using
OpenRouter.

## Trace Contract

Both text and voice mode emit the same trace shapes:

- `voice.utterance_in`,
- `voice.utterance_out`.

The payload includes:

- text,
- turn number,
- mode (`text` or `voice`).

Same trace contract means downstream evaluation can work regardless of whether
audio was used.

## Graceful Degradation

Good voice systems degrade cleanly:

- no Speechmatics key → fall back to text mode,
- no Rime key → print replies instead of speaking,
- microphone failure → explain the system permission issue.

This is better than crashing in a way the user cannot interpret.

## Exam Checklist

You should be able to explain:

1. What STT and TTS mean.
2. Why text mode is still valuable.
3. What the manager persona is responsible for.
4. Why voice mode should emit the same trace events as text mode.
5. Why voice is a transport layer, not the policy brain.
