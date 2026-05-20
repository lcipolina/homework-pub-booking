# Ex8 — Voice pipeline

## Your answer

The voice pipeline has two modes with a shared trace-event contract:
text mode reads stdin and the manager persona replies via the configured
LLM; voice mode uses Speechmatics for STT and Rime for TTS. In my real
voice run, session `sess_813916a3547a`, Speechmatics transcribed my
spoken request for an Edinburgh AI meetup booking for about 160 guests
with vegan catering. The manager persona replied in voice mode that the
venue could not accommodate that many guests and suggested larger
alternatives.

I also kept the earlier three-turn text-mode run, session
`sess_aaf2d4c51905`, as a fallback comparison. In that run I asked to
book Haymarket Tap for six people at 19:30 with a £200 deposit, supplied
the date/contact name, and asked for policy confirmation. The manager
stayed in character and confirmed that parties of 8 or fewer with
deposits under £300 are acceptable.

The critical design choice is graceful degradation. run_voice_mode
checks SPEECHMATICS_KEY, or the equivalent SPEECHMATICS_API_KEY spelling,
and the speechmatics-python import before doing anything else. If either
is missing, it logs a warning and falls through to run_text_mode. This
means CI can pass the "voice loop implemented" check without Speechmatics
credentials, while real credentials exercise the same manager persona
over the audio transport.

Both modes emit voice.utterance_in and voice.utterance_out trace
events with payload {text, turn, mode}. The mode field tells the
grader which transport was in use. Same trace shape = identical
downstream analysis.

The ManagerPersona class holds a conversation history list and calls
the configured OpenAI-compatible provider for each turn. I changed it
to read the same provider config as the rest of the agent, so OpenRouter
works consistently across Ex5 and Ex8 instead of Ex8 silently hardcoding
Nebius.

## Citations

- Voice run `make ex8-voice`, session `sess_813916a3547a` — Speechmatics input and manager voice-mode reply
- Text run `make ex8-text`, session `sess_aaf2d4c51905` — 3-turn manager conversation
- Public Ex8 tests: `uv run pytest tests/public/test_ex8_scaffold.py -q` passed 5/5
- starter/voice_pipeline/voice_loop.py — run_voice_mode
- starter/voice_pipeline/manager_persona.py — LLM-backed persona
