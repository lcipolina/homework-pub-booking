# 04 — Handoff Bridge State Machine

## Overview

Ex7 wires the loop half and structured half together. The important idea is
round-trip control:

```text
loop → structured → complete
loop → structured → loop → structured → complete
```

The second path is the interesting one. It means the structured half can reject
a proposal and send the loop half back to research an alternative.

## Forward Handoff

A forward handoff happens when the loop half has a candidate booking and wants
policy validation.

It contains:

- source half,
- target half,
- reason,
- context,
- booking data,
- return instructions.

The structured half receives the handoff data and runs Rasa validation.

## Rejection and Reverse Task

If Rasa rejects, the bridge creates a new loop task such as:

```text
The structured half rejected the previous proposal.
Reason: party_too_large. Produce an alternative.
```

It also includes machine-readable context:

```python
{
    "prior_result": ...,
    "rejection_reason": "party_too_large",
    "retry": True,
}
```

This is how the loop half learns what to change.

## State Machine

The bridge has bounded rounds:

1. start loop round,
2. if loop completes, finish,
3. if loop hands off, call structured,
4. if structured confirms, finish,
5. if structured escalates, build reverse task,
6. stop after `max_rounds`.

Bounded rounds prevent infinite retries.

## Audit Trail

The bridge writes trace events:

- `bridge.round_start`,
- `session.state_changed`,
- tool calls from loop execution.

It also archives old handoff files instead of silently overwriting them.

## Real-Mode Lesson

In this repo, Ex7 real mode originally had a documentation mismatch: the
Makefile advertised `make ex7-real`, but the target was missing. Adding the
target made the advertised workflow executable.

## Exam Checklist

You should be able to answer:

1. What is a forward handoff?
2. What is a reverse handoff or reverse task?
3. Why is `max_rounds` necessary?
4. What trace events prove the bridge actually ran?
5. Why should rejected policy decisions go back to the loop half?
