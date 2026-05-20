# Study Roadmap — How To Actually Learn This Homework

This roadmap is for understanding, not just submitting.

## Phase 1 — Build The Mental Model

Goal:

> Explain the system without mentioning file names.

Read:

```text
README.md
00_Homework_Guide.md
08_Infographics_and_Glossary.md
```

You should be able to answer:

- Why is one LLM prompt not enough?
- What is the difference between loop half and structured half?
- Why does the system need traces?
- Why is voice just a pipeline around text?

Exercise:

Draw this from memory:

```text
Loop -> Tools -> Integrity
Loop -> Handoff -> Rasa -> approve/reject
Voice -> STT -> Persona -> TTS
```

## Phase 2 — Understand Ex5 Deeply

Read:

```text
02_Loop_Half_Tools_and_Dataflow.md
starter/edinburgh_research/tools.py
starter/edinburgh_research/integrity.py
```

Run:

```bash
make ex5
make ex5-real
```

Questions:

- Which tools are read-only?
- Why is `generate_flyer` not parallel-safe?
- What does `_TOOL_CALL_LOG` store?
- How would the checker catch `£9999`?

Mini challenge:

Open a generated flyer and identify which facts should be traceable to tool
outputs.

## Phase 3 — Understand Rasa And Policy

Read:

```text
03_Rasa_CALM_Structured_Half.md
starter/rasa_half/validator.py
rasa_project/actions/actions.py
```

Run:

```bash
make ex6
make ex6-auto
```

Questions:

- What is a slot?
- Why normalize currency before policy validation?
- Why should `party_size="abc"` fail instead of becoming zero?
- Why is Rasa better than an LLM prompt for deposit rules?

Mini challenge:

Trace one payload from raw booking fields to approval/rejection.

## Phase 4 — Understand The Bridge

Read:

```text
04_Handoff_Bridge_State_Machine.md
starter/handoff_bridge/bridge.py
answers/ex9_reflection.md
```

Run:

```bash
make ex7
make ex7-real
```

Questions:

- What is a forward handoff?
- What is a reverse handoff?
- Why must rejection reason be preserved?
- What stops infinite retry?

Mini challenge:

Find the trace event where structured sends control back to loop.

## Phase 5 — Understand Voice

Read:

```text
05_Voice_Pipeline_STT_TTS.md
starter/voice_pipeline/voice_loop.py
starter/voice_pipeline/manager_persona.py
```

Run:

```bash
make ex8-text
make ex8-voice
```

Questions:

- What does STT produce?
- What does TTS consume?
- Why did 160 guests correctly trigger rejection?
- Why is `mode: voice` important in trace events?

Mini challenge:

Run one accepted voice scenario:

```text
"Hi Alasdair, I want to book six people tonight at 7:30 with a deposit of 200 pounds."
```

## Phase 6 — Understand Providers

Read:

```text
06_Provider_Config_OpenRouter_Nebius.md
.env
rasa_project/endpoints.yml
```

Run:

```bash
make verify
```

Questions:

- What is a base URL?
- What is an API key?
- Why should `.env` never be committed?
- What changed when we used OpenRouter instead of Nebius?

## Phase 7 — Understand Evaluation

Read:

```text
07_Evaluation_Logs_and_Reflection.md
docs/grading-rubric.md
answers/ex9_reflection.md
```

Run:

```bash
make check-submit
make ci
```

Questions:

- Why does local show `0/30` reasoning?
- What makes an Ex9 answer grounded?
- What exact artifact proves Ex7 recovered from rejection?
- What exact artifact proves Ex8 voice happened?

## Mastery Checklist

You understand the homework if you can explain:

- [ ] why tool outputs are more trustworthy than LLM prose,
- [ ] why dataflow integrity catches unsupported flyer facts,
- [ ] why Rasa owns booking policy,
- [ ] how bridge state transitions avoid fake completion,
- [ ] how voice mode is audio transport around text reasoning,
- [ ] why sessions prevent cross-run evidence contamination,
- [ ] how to read `trace.jsonl`,
- [ ] how to cite a ticket in Ex9.

## One-Hour Review Plan

If you have only one hour:

```text
10 min  Read 08 diagrams/glossary.
15 min  Read Ex5 dataflow chapter.
10 min  Read Rasa validation chapter.
10 min  Read handoff state machine chapter.
10 min  Read voice chapter.
5 min   Read Ex9 rubric notes.
```

## Three-Hour Deep Plan

```text
45 min  Ex5 tools and integrity
35 min  Rasa structured half
35 min  Handoff bridge
25 min  Voice pipeline
20 min  Provider config
20 min  Evaluation and Ex9
20 min  Re-run commands and inspect traces
```

## Best Final Exercise

Explain the whole homework out loud using this sentence frame:

```text
"The loop half is allowed to explore, but every important fact must come from
a tool. When the loop proposes a booking, Rasa validates policy. If Rasa
rejects, the bridge sends the rejection reason back to the loop. Voice mode
wraps the same manager logic with STT and TTS. Sessions, traces, and tickets
make all of this auditable."
```

If that sentence makes sense, the architecture has clicked.

