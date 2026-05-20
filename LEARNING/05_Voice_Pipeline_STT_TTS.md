# 05 — Voice Pipeline: STT, Agent, TTS

## Voice Agents Are Event Pipelines

Voice feels different from text, but the underlying architecture is familiar:

```text
input -> model/system -> output
```

Text mode:

```text
keyboard text ──▶ manager persona ──▶ terminal text
```

Voice mode:

```text
microphone audio ──▶ speech-to-text ──▶ manager persona ──▶ text-to-speech ──▶ speaker audio
```

The manager persona does not need to know whether the user typed or spoke. It
receives text either way.

## The Full Ex8 Pipeline

```text
┌──────────────┐
│ Microphone   │
│ raw audio    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Speechmatics │
│ STT          │
└──────┬───────┘
       │ transcript
       ▼
┌──────────────────┐
│ Manager Persona  │
│ LLM conversation │
└──────┬───────────┘
       │ reply text
       ▼
┌──────────────┐
│ Rime         │
│ TTS          │
└──────┬───────┘
       │ audio
       ▼
┌──────────────┐
│ Speakers     │
└──────────────┘
```

## STT: Speech-To-Text

Speech-to-text converts audio into text.

In this homework:

```text
Speechmatics
```

does the STT work.

Input:

```text
audio waveform from microphone
```

Output:

```text
"Hi Alasdair, I would like to book a table for six people..."
```

The agent cannot reason over raw audio. It reasons over the transcript.

## TTS: Text-To-Speech

Text-to-speech converts text into audio.

In this homework:

```text
Rime
```

does the TTS work.

Input:

```text
"Aye, six people at half seven is fine..."
```

Output:

```text
audio played through speakers
```

## Why The Assignment Mentions Voice But Tests Text Too

Voice systems have more external dependencies:

- microphone permissions,
- audio devices,
- STT API key,
- TTS API key,
- network,
- codecs,
- timing/silence detection.

The homework therefore provides text mode as a reliable fallback.

```text
Text mode:
  lower external complexity
  still tests persona and trace events

Voice mode:
  real integration
  more realistic
  more fragile
```

## Graceful Degradation

Graceful degradation means:

> If a real voice dependency is missing, the system should still run in a
> simpler mode instead of crashing.

In Ex8:

```text
if Speechmatics key missing:
  warn user
  fall back to text mode
```

This is good engineering. It lets CI and local development test the logic even
when audio credentials are unavailable.

## Silence Detection

Voice mode needs to know when your turn is done.

In a text chat, pressing Enter sends the message.

In voice, the system needs a rule:

```text
If silence lasts 2 seconds, end the user's turn.
```

That is why the console says:

```text
Silence for 2.0s ends a turn.
```

## The Manager Persona

The manager persona is an LLM with a system prompt.

It should behave like:

- a gruff Edinburgh pub manager,
- policy-aware,
- not too permissive,
- able to reject with reasons.

Key rules:

```text
Accept if:
  party_size <= 8
  deposit <= £300

Decline otherwise.
```

So when we said:

```text
"160 guests"
```

the correct behavior was rejection.

## Voice Trace Events

Voice mode records the same conceptual events as text mode:

```json
{
  "event_type": "voice.utterance_in",
  "payload": {
    "text": "transcribed user speech",
    "turn": 0,
    "mode": "voice"
  }
}
```

```json
{
  "event_type": "voice.utterance_out",
  "payload": {
    "text": "manager reply",
    "turn": 0,
    "mode": "voice"
  }
}
```

Why this matters:

> The grader and humans can inspect the conversation even though it happened
> through audio.

## Our Real Voice Run

The real run proved:

```text
Speechmatics transcribed user speech.
Manager persona responded.
Trace recorded mode="voice".
```

The manager rejected 160 guests, which is correct because 160 is above the
policy limit.

That run demonstrates the transport pipeline, not just the text fallback.

## Common Voice Failure Modes

### 1. Wrong Environment Variable Name

The homework expected:

```text
SPEECHMATICS_KEY
```

The key was added as:

```text
SPEECHMATICS_API_KEY
```

We added compatibility so either spelling works.

### 2. Microphone Permission

macOS may block terminal microphone access.

Fix:

```text
System Settings -> Privacy & Security -> Microphone
```

Allow your terminal app.

### 3. Silence Too Soon

If you do not speak after:

```text
[turn 1] listening...
```

the system may end the conversation.

### 4. STT Mishears You

Speech recognition can produce imperfect text. The agent sees the transcript,
not what you intended.

Example:

```text
"Hi Alasdair" -> "Hello unless there"
```

That is a normal voice-agent problem. Good systems handle imperfect transcripts.

## Repo Files To Study

- `starter/voice_pipeline/voice_loop.py`
  - text mode,
  - voice mode,
  - STT/TTS integration,
  - trace events.

- `starter/voice_pipeline/manager_persona.py`
  - system prompt,
  - provider call,
  - conversation history.

- `starter/voice_pipeline/run.py`
  - command-line entrypoint.

- `tests/public/test_ex8_scaffold.py`
  - public checks for persona and fallback behavior.

## The Big Lesson

Voice agents are not fundamentally different from text agents. They add
translation layers around the same reasoning core.

```text
Audio in -> text -> agent -> text -> audio out
```

Once you see that, voice becomes less mysterious and more debuggable.

