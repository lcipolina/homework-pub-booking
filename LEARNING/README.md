# Week 5 Pub Booking — Deep Learning Guide

This folder is a conceptual textbook for the Week 5 homework. It is written
from the top: not "which blank do I fill," but "what kind of system am I
building, why is it split this way, and what does each file teach?"

The homework is a miniature production agent system. It combines:

- an LLM loop that can research, call tools, and recover from mistakes,
- deterministic tools that provide grounded facts,
- an integrity checker that catches unsupported facts,
- a Rasa CALM structured dialog half that enforces booking policy,
- a bridge that moves control between flexible reasoning and strict policy,
- a voice layer that turns typed dialog into microphone and speaker dialog,
- traces, tickets, and session directories that make the whole run auditable.

## How To Use This Folder

Read these in order if you want the deep version:

1. `00_Homework_Guide.md`
   - The whole assignment from first principles.
   - What Ex5-Ex9 are really teaching.
   - How to read the repo as a system, not a pile of scripts.

2. `01_Sovereign_Agent_Architecture.md`
   - The architecture vocabulary: sessions, traces, tickets, halves, tools.
   - Why agent systems need durable evidence.
   - How `sovereign-agent` packages production patterns.

3. `02_Loop_Half_Tools_and_Dataflow.md`
   - What an agent "tool" is.
   - Why tools need contracts.
   - How Ex5 catches LLM fabrication with dataflow integrity.

4. `03_Rasa_CALM_Structured_Half.md`
   - Why Rasa exists next to an LLM.
   - What CALM/dialog managers do.
   - How validation differs from generation.

5. `04_Handoff_Bridge_State_Machine.md`
   - Why systems hand off between components.
   - How a rejection becomes a new loop task.
   - How state machines prevent vague "agent magic."

6. `05_Voice_Pipeline_STT_TTS.md`
   - How microphone audio becomes text.
   - How text becomes speech.
   - Why voice agents are still just event pipelines.

7. `06_Provider_Config_OpenRouter_Nebius.md`
   - What API keys, model providers, base URLs, and env vars are.
   - Why OpenRouter worked here when Nebius did not.
   - How provider abstraction protects your code.

8. `07_Evaluation_Logs_and_Reflection.md`
   - How the grader thinks.
   - Why Ex9 is about evidence, not essay-writing.
   - How to cite traces and tickets convincingly.

9. `08_Infographics_and_Glossary.md`
   - One-page diagrams and glossary.
   - Use this when a term feels slippery.

10. `09_Output_Review_Guide.md`
    - Where conclusions are written.
    - Where to inspect model outputs, traces, and consistency evidence.

11. `Appendix_Commands.md`
    - Commands, diagnostics, common failure modes.

12. `Study_Roadmap.md`
    - A structured study plan for mastering the homework.

## Big Picture Diagram

```text
                         ┌──────────────────────────────┐
                         │      User / Homework Task     │
                         │  "Book a pub in Edinburgh"    │
                         └───────────────┬──────────────┘
                                         │
                                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         Sovereign Agent Session                      │
│                                                                     │
│  ┌──────────────────────┐       handoff       ┌──────────────────┐  │
│  │ Flexible Loop Half    │ ─────────────────▶ │ Structured Half   │  │
│  │ - planner             │                    │ Rasa CALM         │  │
│  │ - executor            │ ◀───────────────── │ policy validation │  │
│  │ - tool use            │   rejection/retry  │ confirm/reject    │  │
│  └──────────┬───────────┘                    └────────┬─────────┘  │
│             │                                         │            │
│             ▼                                         ▼            │
│  ┌──────────────────────┐                    ┌──────────────────┐  │
│  │ Tools / Ground Truth  │                    │ Booking Result    │  │
│  │ venue, weather, cost  │                    │ approved/rejected │  │
│  └──────────┬───────────┘                    └──────────────────┘  │
│             │                                                      │
│             ▼                                                      │
│  ┌──────────────────────┐                                          │
│  │ Dataflow Integrity    │  "Did the flyer only use tool facts?"    │
│  └──────────────────────┘                                          │
│                                                                     │
│  Everything is recorded in traces, tickets, and session directories.│
└─────────────────────────────────────────────────────────────────────┘
                                         │
                                         ▼
                         ┌──────────────────────────────┐
                         │ Voice Layer, optional Ex8     │
                         │ STT → LLM persona → TTS       │
                         └──────────────────────────────┘
```

## The Core Lesson

An LLM by itself is not a reliable software system. It can reason, but it can
also invent, forget, overgeneralize, and claim success without proof.

This homework teaches how to wrap an LLM in architecture:

- **Tools** provide facts.
- **Schemas** shape data.
- **Structured dialog** enforces policy.
- **State machines** control handoffs.
- **Traces** make behavior inspectable.
- **Integrity checks** make claims testable.

That is the difference between "chatbot that sounds plausible" and "agent
system that can be debugged, graded, and trusted within a limited domain."
