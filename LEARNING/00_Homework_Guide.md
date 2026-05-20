# 00 — Homework Guide From First Principles

## What Are We Building?

The assignment says: build an AI agent that books a pub.

That sounds simple, but the hidden teaching goal is larger:

> Build a system where an LLM can reason flexibly, but where important
> decisions are grounded, validated, logged, and recoverable.

The pub-booking story is a wrapper around common production AI problems:

- How does an agent find information?
- How does it avoid inventing facts?
- How does it know when a policy says "no"?
- How does it recover from rejection?
- How do we inspect what happened after the run?
- How do we expose the system through voice?

## The Human Story

Imagine you are organizing an Edinburgh AI meetup.

You need:

- a pub near Haymarket,
- enough capacity,
- a cost estimate,
- weather context,
- a flyer,
- a booking decision,
- an alternative if the manager rejects,
- maybe a voice call with the manager.

A human assistant would naturally split this into different kinds of work:

```text
Researcher:
  "Which venues are possible?"

Accountant:
  "What will catering cost?"

Designer:
  "Make the flyer."

Policy/Manager:
  "Can we actually accept this booking?"

Coordinator:
  "If rejected, go back and find another plan."

Phone operator:
  "Talk to the manager out loud."
```

The homework turns those roles into software components.

## The Technical Story

The repo implements five exercises:

```text
Ex5: Loop half + tools + dataflow integrity
Ex6: Rasa structured half
Ex7: Handoff bridge between loop and structured halves
Ex8: Voice pipeline
Ex9: Reflection grounded in run logs
```

Each exercise adds one architectural ability.

```text
┌─────┬───────────────────────────────┬──────────────────────────────┐
│ Ex  │ User-facing story             │ Architecture lesson           │
├─────┼───────────────────────────────┼──────────────────────────────┤
│ 5   │ Research pubs and make flyer  │ Tool use + integrity          │
│ 6   │ Confirm/reject booking        │ Structured dialog + policy    │
│ 7   │ Recover when rejected         │ State machine + handoff       │
│ 8   │ Speak to pub manager          │ STT/TTS event pipeline        │
│ 9   │ Explain what happened         │ Evidence-based reflection     │
└─────┴───────────────────────────────┴──────────────────────────────┘
```

## Why Not Just Ask One LLM?

You could ask:

> "Find a pub, estimate cost, make a flyer, and book it."

The LLM might answer beautifully, but there are problems:

1. It may invent venues.
2. It may invent prices.
3. It may ignore policy.
4. It may say a booking is confirmed when it is not.
5. It may be impossible to debug why it chose something.

This homework prevents those failure modes by making the system explicit.

```text
Unstructured chatbot:

  prompt ──▶ LLM ──▶ plausible answer

Problem:
  The answer may sound right even when unsupported.

Agent architecture:

  prompt ──▶ planner ──▶ tool calls ──▶ trace ──▶ validator ──▶ answer
                         ▲             │
                         └──── integrity check

Benefit:
  Claims can be traced back to evidence.
```

## The Three Kinds Of Intelligence In The Homework

### 1. Generative Intelligence

The LLM can interpret a task, plan steps, write natural text, recover from
errors, and hold a conversation.

This appears in:

- Ex5 real-mode executor,
- Ex8 manager persona,
- Ex9 written explanations.

Strength:

- flexible,
- language-native,
- good at vague tasks.

Weakness:

- can fabricate,
- can ignore constraints,
- can be hard to reproduce exactly.

### 2. Procedural Intelligence

Python tools and validators do exact operations:

- filter venues,
- calculate catering cost,
- parse currency,
- check deposit limit,
- write a flyer file.

Strength:

- deterministic,
- testable,
- precise.

Weakness:

- less flexible,
- must be coded in advance.

### 3. Dialog / Policy Intelligence

Rasa decides whether a booking request satisfies policy.

It is not just "chat." It is a controlled dialog manager with slots, flows,
actions, and validation rules.

Strength:

- good at stateful business rules,
- auditable,
- less likely to improvise policy.

Weakness:

- needs structured inputs,
- requires setup and training.

## The Central Split: Loop Half vs Structured Half

The homework uses two halves because one component should not do everything.

```text
┌──────────────────────────────────────┐
│ Loop Half                            │
│                                      │
│ Good for:                            │
│ - research                           │
│ - open-ended planning                │
│ - trying alternatives                │
│ - using tools                        │
│                                      │
│ Bad for:                             │
│ - final policy enforcement           │
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│ Structured Half                      │
│                                      │
│ Good for:                            │
│ - strict booking rules               │
│ - slot validation                    │
│ - confirm/reject decisions           │
│                                      │
│ Bad for:                             │
│ - exploratory research               │
└──────────────────────────────────────┘
```

This is a general pattern in agent systems:

> Let the LLM explore. Let structured code decide.

## The Dataflow Lesson

Ex5 asks the agent to make a flyer. Flyers are dangerous because they are
polished output. Polished output can hide unsupported facts.

Example:

```text
Tool actually returned:
  deposit: £320

Flyer says:
  deposit: £200
```

That is not a formatting issue. It is a truth issue.

So Ex5 records tool outputs, then checks whether flyer facts appear in those
outputs.

```text
tool output ──▶ _TOOL_CALL_LOG ──▶ verify_dataflow ──▶ pass/fail
      │                                      ▲
      └──────────── flyer claims ───────────┘
```

The phrase "dataflow integrity" means:

> Important claims in the final artifact must be traceable to earlier
> trusted data.

## The Handoff Lesson

Ex7 demonstrates recovery.

The loop proposes a booking. The structured half rejects. The bridge turns
that rejection into a new loop task.

```text
Loop:
  "Try booking 12 people."

Structured:
  "Rejected: party_too_large."

Bridge:
  "Loop, try again with that rejection reason."

Loop:
  "Try booking 6 people at another venue."

Structured:
  "Approved."
```

This is much more robust than asking the LLM to "remember to retry." The
retry is encoded in the state machine.

## The Voice Lesson

Voice mode sounds magical, but conceptually it is just a transport change.

Text mode:

```text
keyboard text ──▶ manager persona ──▶ terminal text
```

Voice mode:

```text
microphone audio ──▶ STT ──▶ manager persona ──▶ TTS ──▶ speaker audio
```

The same manager policy can run in both modes. The system logs both as
`voice.utterance_in` and `voice.utterance_out`.

## What "Done" Means

The homework is not done when the code merely runs once. It is done when:

- public tests pass,
- local grader passes,
- real sessions exist,
- traces prove what happened,
- Ex9 cites evidence,
- no secrets are committed,
- the repo is pushed to your fork.

The local check showed:

```text
Mechanical: 27/27
Behavioural: 19/19
Reasoning: CI-only
```

That means all locally measurable work is complete. The remaining reasoning
points are judged by CI from your written answers and session evidence.

