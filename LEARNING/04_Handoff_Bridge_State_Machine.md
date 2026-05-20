# 04 — Handoff Bridge And State Machines

## Why A Bridge Exists

Ex5 has a loop half. Ex6 has a structured half. Ex7 asks:

> How do these two halves cooperate?

The answer is the handoff bridge.

The bridge is the coordinator. It decides which half runs next, passes data
between halves, and prevents rejected work from being treated as success.

## The Core Problem

Suppose the loop half proposes:

```text
Book 12 people at Haymarket Tap.
```

The structured half says:

```text
Rejected: party_too_large.
```

What should happen?

Bad system:

```text
LLM: "Okay, booking completed."
```

Good system:

```text
Bridge: "Loop, the structured half rejected this. Try an alternative that
fits the policy."
```

The bridge makes rejection actionable.

## What Is A Handoff?

A handoff is a controlled transfer of responsibility from one component to
another.

```text
Loop half ──handoff payload──▶ Structured half
```

The payload must include enough context for the next half:

```json
{
  "reason": "candidate found; confirm under policy",
  "context": "party of 12 near Haymarket",
  "data": {
    "action": "confirm_booking",
    "venue_id": "Haymarket Tap",
    "party_size": "12",
    "deposit": "£0"
  }
}
```

## Forward And Reverse Handoffs

Forward handoff:

```text
Loop ──▶ Structured
```

Used when the loop has a proposal and wants policy confirmation.

Reverse handoff:

```text
Structured ──▶ Loop
```

Used when the structured half rejects and the loop needs to research again.

## Ex7 State Machine

```text
                  ┌────────────┐
                  │ Start      │
                  └─────┬──────┘
                        │
                        ▼
                  ┌────────────┐
                  │ Loop       │
                  │ research   │
                  └─────┬──────┘
                        │ handoff
                        ▼
                  ┌────────────┐
                  │ Structured │
                  │ validate   │
                  └─────┬──────┘
             approve    │    reject
        ┌───────────────┘    └──────────────┐
        ▼                                    ▼
  ┌────────────┐                       ┌────────────┐
  │ Complete   │                       │ Reverse    │
  │ success    │                       │ handoff    │
  └────────────┘                       └─────┬──────┘
                                             │
                                             ▼
                                      ┌────────────┐
                                      │ Loop retry │
                                      └────────────┘
```

This is a state machine because only certain transitions are allowed.

## Why State Machines Matter

LLM text can be ambiguous:

```text
"Looks like we should probably try another venue."
```

A state machine is explicit:

```json
{
  "from": "structured",
  "to": "loop",
  "rejection_reason": "party_too_large"
}
```

That is testable. The grader can check it. Humans can debug it.

## The Ex7 Real Run

The important trace looked like this:

```text
round 1:
  loop searches party_size=12
  loop calls handoff_to_structured
  bridge moves loop -> structured
  structured rejects party_too_large
  bridge moves structured -> loop

round 2:
  loop searches party_size=6
  loop calls handoff_to_structured
  bridge moves loop -> structured
  structured approves
  bridge moves structured -> complete
```

That is the whole homework architecture in miniature:

```text
try -> validate -> reject -> retry -> validate -> complete
```

## Why The Rejection Reason Must Be Preserved

If the bridge only says:

```text
"Try again."
```

the loop might repeat the same mistake.

If the bridge says:

```text
"Rejected because party_too_large."
```

the loop can change the relevant variable.

```text
party_size=12  -> rejected
party_size=6   -> approved
```

That is why reverse task construction matters.

## Handoff Files

The bridge uses files as IPC.

Conceptually:

```text
ipc/handoff_to_structured.json
```

Why files?

- easy to inspect,
- durable,
- language-agnostic,
- simple for homework.

But visible handoff files must not pile up. If there are multiple active
handoff files, the system becomes ambiguous:

```text
handoff_A.json
handoff_B.json

Which one is current?
```

So stale handoffs are archived.

## Max Rounds

The bridge should not loop forever.

```text
round 1: rejected
round 2: rejected
round 3: rejected
...
```

`max_rounds` gives the system a boundary. If the structured half keeps
rejecting, the bridge reports failure instead of spinning.

This is a production safety concept:

> Every autonomous loop needs a stopping rule.

## Common Mistakes

### Mistake 1: Treating Rejection As Final Failure

The point of Ex7 is recovery. Rejection should trigger a reverse handoff, not
immediate abandonment, unless max rounds are exhausted.

### Mistake 2: Losing The Handoff Payload

If the bridge drops fields like `party_size`, `venue_id`, or
`rejection_reason`, the next half cannot make a grounded decision.

### Mistake 3: Letting Planner Text Control Everything

Planner prose is useful context. State transitions are the control plane.

## Repo Files To Study

- `starter/handoff_bridge/bridge.py`
  - bridge loop,
  - forward handoff,
  - reverse task,
  - stale handoff archival.

- `starter/handoff_bridge/run.py`
  - demo scenario,
  - real Rasa lifecycle support.

- `starter/rasa_half/structured_half.py`
  - structured half called by bridge.

- session trace:
  - `logs/trace.jsonl`
  - records state changes.

## The Big Lesson

Agents need orchestration, not just prompts.

```text
Prompt:
  "Try again if rejected."

Bridge:
  structured rejected -> build retry task -> run loop again
```

The second one is reliable because it is encoded in the system, not merely
requested in natural language.

