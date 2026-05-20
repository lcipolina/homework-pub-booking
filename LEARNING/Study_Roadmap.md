# Study Roadmap

## Goal

Use this plan to understand the homework deeply enough to explain it in an
interview, exam, or written reflection.

## Day 1 — Architecture Overview

Read:

- `00_Homework_Guide.md`
- `01_Sovereign_Agent_Architecture.md`

Practice explaining:

- session,
- ticket,
- trace,
- loop half,
- structured half,
- handoff.

## Day 2 — Ex5 Loop Half

Read:

- `02_Loop_Half_Tools_and_Dataflow.md`

Then open:

- `starter/edinburgh_research/tools.py`
- `starter/edinburgh_research/integrity.py`
- `starter/edinburgh_research/run.py`

Run:

```bash
make ex5
make narrate SESSION=<session_id>
```

Practice explaining why `£9999` should fail dataflow validation.

## Day 3 — Ex6 Rasa Structured Half

Read:

- `03_Rasa_CALM_Structured_Half.md`

Then open:

- `starter/rasa_half/validator.py`
- `starter/rasa_half/structured_half.py`
- `rasa_project/actions/actions.py`
- `rasa_project/data/flows.yml`

Run:

```bash
make ex6
make ex6-auto
```

Practice explaining the difference between validation in Python and language
understanding in Rasa.

## Day 4 — Ex7 Bridge

Read:

- `04_Handoff_Bridge_State_Machine.md`

Then open:

- `starter/handoff_bridge/bridge.py`
- `starter/handoff_bridge/run.py`

Run:

```bash
make ex7
make ex7-real
```

Practice drawing the state machine from memory.

## Day 5 — Ex8 Voice and Provider Config

Read:

- `05_Voice_Pipeline_STT_TTS.md`
- `06_Provider_Config_OpenRouter_Nebius.md`

Then open:

- `starter/voice_pipeline/voice_loop.py`
- `starter/voice_pipeline/manager_persona.py`
- `.env`
- `rasa_project/endpoints.yml`

Practice explaining why voice is a transport layer.

## Day 6 — Reflection and Submission

Read:

- `07_Evaluation_Logs_and_Reflection.md`
- `Appendix_Commands.md`

Run:

```bash
uv run pytest tests/public -q
make check-submit
```

Then review each answer file and ask:

- Did I cite a real session?
- Did I describe actual tool calls?
- Did I explain a real failure or design tradeoff?

## Final Self-Test

Answer these without notes:

1. Why does the homework use both loop and structured halves?
2. What exactly does dataflow integrity verify?
3. Why is Rasa better than a prompt for deposit/party-size policy?
4. What is a reverse handoff?
5. Why can real mode fail when offline mode passes?
6. What changes when using OpenRouter instead of Nebius?
