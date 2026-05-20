# Week 5 Pub Booking — Learning Textbook

This folder explains the concepts behind the Week 5 homework in the same
style as your Week 1 `LEARNING` folder.

The homework is not only about making scripts pass. It teaches a production
agent architecture:

1. a flexible loop half that researches and uses tools,
2. a structured half that enforces booking policy,
3. a bridge that moves control between them,
4. a voice layer that turns the agent into a real-time conversation,
5. integrity checks that catch unsupported or fabricated facts.

## Chapter Map

- `00_Homework_Guide.md` — what the whole homework is building.
- `01_Sovereign_Agent_Architecture.md` — sessions, tickets, traces, halves.
- `02_Loop_Half_Tools_and_Dataflow.md` — Ex5, tools, flyer, integrity.
- `03_Rasa_CALM_Structured_Half.md` — Ex6, Rasa, validation, policies.
- `04_Handoff_Bridge_State_Machine.md` — Ex7, loop to structured and back.
- `05_Voice_Pipeline_STT_TTS.md` — Ex8, speech-to-text and text-to-speech.
- `06_Provider_Config_OpenRouter_Nebius.md` — keys, models, OpenRouter switch.
- `07_Evaluation_Logs_and_Reflection.md` — Ex9, grading, evidence.
- `Appendix_Commands.md` — practical commands and troubleshooting.
- `Study_Roadmap.md` — a study plan for mastering the homework.

## Learning Outcomes

After reading this folder, you should be able to explain:

- why flexible agent loops need deterministic tool contracts,
- why booking confirmation belongs in a structured/policy half,
- how a reverse handoff recovers from rejection,
- why dataflow integrity catches LLM fabrication,
- how real-mode failures differ from code bugs,
- how to ground written answers in trace evidence.
