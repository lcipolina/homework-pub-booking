# 08 — Infographics And Glossary

This chapter is a visual reference. Use it when the architecture words blur
together.

## One-Screen Architecture

```text
USER TASK
  │
  ▼
LOOP HALF
  planner creates subgoals
  executor calls tools
  tools return facts
  │
  ├── Ex5: flyer + dataflow check
  │
  └── Ex7: handoff proposal
           │
           ▼
STRUCTURED HALF
  Rasa receives booking payload
  validators normalize fields
  action accepts/rejects
           │
           ├── approve ──▶ complete
           │
           └── reject ──▶ bridge builds retry task ──▶ LOOP HALF

VOICE LAYER
  microphone -> STT -> manager persona -> TTS -> speakers

OBSERVABILITY
  sessions, traces, tickets, tool logs, artifacts
```

## Ex5 Dataflow

```text
fixtures/*.json
     │
     ▼
tools.py
  venue_search
  get_weather
  calculate_cost
     │
     ▼
_TOOL_CALL_LOG
     │
     ├──────────────┐
     ▼              ▼
generate_flyer   verify_dataflow
     │              ▲
     ▼              │
workspace/flyer.html
```

Meaning:

> The flyer is not trusted until its important facts are checked against tool
> outputs.

## Ex6 Rasa Validation

```text
Raw booking payload
  party_size="12"
  deposit="£500"
        │
        ▼
normalise_booking_payload
        │
        ▼
Typed values
  party_size=12
  deposit_gbp=500
        │
        ▼
ActionValidateBooking
        │
        ├─ party_size > 8  -> reject party_too_large
        ├─ deposit > 300   -> reject deposit_too_high
        └─ otherwise       -> approve
```

## Ex7 Handoff Loop

```text
Round 1
  Loop proposes party=12
       │
       ▼
  Structured rejects party_too_large
       │
       ▼
  Bridge creates retry task

Round 2
  Loop proposes party=6
       │
       ▼
  Structured approves
       │
       ▼
  Complete
```

## Ex8 Voice

```text
You speak
   │
   ▼
Microphone
   │ audio
   ▼
Speechmatics STT
   │ transcript
   ▼
Manager persona LLM
   │ reply text
   ▼
Rime TTS
   │ audio
   ▼
You hear reply
```

## Core Terms

### Agent

A system that can take a task, decide actions, use tools, and produce results.

### LLM

Large language model. It predicts/generates text, but can also be used for
planning, summarizing, and conversation.

### Tool

A function exposed to the agent. Tools provide controlled abilities such as
searching fixtures, calculating cost, or generating a file.

### Planner

The component that breaks a task into subgoals.

### Executor

The component that carries out subgoals, usually by calling tools.

### Session

One isolated run of the system, with its own logs and workspace.

### Trace

An event log recording what happened during the run.

### Ticket

A durable record of a planner or executor unit of work.

### Artifact

A file produced by the run, such as `flyer.html`.

### Dataflow

The path facts take from trusted sources through tools into final output.

### Integrity Check

A check that verifies important final claims are supported by trusted earlier
data.

### Fabrication

An unsupported generated claim. It may sound plausible, but it is not grounded
in the run's evidence.

### Structured Half

The strict policy/dialog component, implemented with Rasa in this homework.

### Loop Half

The flexible planning/tool-use component.

### Handoff

A transfer of control and data from one component to another.

### Reverse Handoff

A handoff from structured half back to loop half after rejection.

### STT

Speech-to-text. Converts audio to text.

### TTS

Text-to-speech. Converts text to audio.

### Environment Variable

A configuration value read by programs at runtime, often used for secrets.

### Provider

A company/service hosting a model API, such as OpenRouter, Nebius,
Speechmatics, or Rime.

## Concept Pairs

```text
Flexible vs strict
  Loop half vs structured half

Generated vs grounded
  LLM prose vs tool output

Claim vs evidence
  flyer text vs _TOOL_CALL_LOG

Conversation vs policy
  manager persona vs validation action

Local vs CI
  public checks vs hidden/reasoning grader
```

## The Tiny Summary

```text
Agents need freedom to solve tasks.
Production systems need constraints to trust results.
This homework teaches how to combine both.
```

