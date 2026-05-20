# 07 — Evaluation, Logs, And Reflection

## The Grader Is Testing Architecture

The grader is not only checking whether a function returns something. It is
checking whether the system has the right architecture:

- tools exist,
- tools log facts,
- integrity catches fabrication,
- Rasa validates policy,
- bridge recovers from rejection,
- voice events are traced,
- written reflections cite evidence.

That is why a nice final output is not enough.

## The Three Scoring Layers

```text
┌────────────────┬────────┬──────────────────────────────────────────┐
│ Layer          │ Points │ What it checks                            │
├────────────────┼────────┼──────────────────────────────────────────┤
│ Mechanical     │ 27/30* │ files, formatting, tests, imports         │
│ Behavioural    │ 19/40* │ scenarios run locally                     │
│ Reasoning      │ 30     │ Ex9, judged in CI                         │
└────────────────┴────────┴──────────────────────────────────────────┘
```

The local `make check-submit` in this repo reports 46/76 because only 46
points are locally measurable in the public grader.

Important:

```text
0/30 reasoning locally does not mean lost.
It means not locally scored.
```

CI reads the Ex9 answers and session artifacts.

## What Logs Are For

Logs are not decoration. They are evidence.

A good Ex9 answer says:

```text
"In session sess_3adc43074c06, trace.jsonl records structured -> loop with
rejection_reason party_too_large."
```

A weak answer says:

```text
"The bridge handled rejection."
```

The first is grounded. The second may be true, but it is not proven.

## Evidence Pyramid

```text
Strongest
  │
  ├─ Exact trace/ticket path + exact field/value
  │    logs/trace.jsonl: rejection_reason=party_too_large
  │
  ├─ Session id + event summary
  │    sess_3adc43074c06 rejected round 1 and approved round 2
  │
  ├─ General file reference
  │    bridge.py handles handoff
  │
  └─ Vague statement
       "It works"

Weakest
```

## Ex9 Question 1

Question theme:

> Planner/handoff decision.

What the judge wants:

- real Ex7 logs,
- specific subgoal,
- `assigned_half`,
- explanation of how the bridge used the decision.

Good evidence:

```text
session: sess_3adc43074c06
ticket: logs/tickets/tk_5b8dcd34/raw_output.json
field: assigned_half = "loop"
trace: loop -> structured -> loop, rejection_reason=party_too_large
```

Concept:

> Planner assignment starts work, but bridge state transitions control the
> system outcome.

## Ex9 Question 2

Question theme:

> Dataflow integrity.

What the judge wants:

- a specific reproducible integrity-check scenario,
- not a generic statement that "integrity is important."

Good scenario:

```text
1. Run Ex5.
2. Tool returns real costs/weather.
3. Modify flyer to include £9999.
4. verify_dataflow fails because £9999 was not in tool outputs.
```

Concept:

> The checker should detect unsupported facts, even if they look plausible.

## Ex9 Question 3

Question theme:

> Pick one framework primitive and one failure mode.

The wording matters:

```text
Exactly one primitive.
Exactly one failure mode.
```

Good:

```text
Primitive: session directories.
Failure mode: cross-run evidence contamination.
```

Bad:

```text
"I would keep sessions, tickets, traces, and memory because many problems..."
```

That may be thoughtful, but it violates the rubric.

## Word Counts

The reasoning rubric expects each answer to be 100-400 words.

Too short:

- not enough evidence,
- likely shallow.

Too long:

- violates constraint,
- judge may penalize.

Our current Ex9 answers were checked:

```text
Q1: 210 words
Q2: 151 words
Q3: 150 words
```

## Local vs CI

Local:

```text
make test
make ci
make check-submit
```

CI:

```text
hidden tests
LLM-as-judge reasoning
possibly real integration checks
```

Local green is necessary but not a guarantee of full CI points.

However, local green plus strong evidence-based Ex9 answers is the right
submission posture.

## How To Read A Trace

A trace is JSON Lines:

```text
one JSON object per line
```

You can search it:

```bash
rg "party_too_large" logs/trace.jsonl
rg "voice.utterance" logs/trace.jsonl
rg "session.state_changed" logs/trace.jsonl
```

Look for:

- event type,
- actor,
- timestamp,
- payload.

## How To Read Tickets

Tickets live under:

```text
logs/tickets/<ticket_id>/
```

Usually useful files:

```text
summary.md       human-readable summary
raw_output.json  exact structured output
manifest.json    metadata
state.json       status
```

If you need exact fields, use `raw_output.json`.

## Why Reflection Is Part Of Engineering

Ex9 is not busywork. It teaches postmortem thinking.

Real agent work often looks like:

```text
Something unexpected happened.
Find the trace.
Find the tool call.
Find the state transition.
Explain the cause.
Patch the architecture.
```

The ability to explain behavior from logs is part of building reliable AI
systems.

## The Big Lesson

An agent is only as trustworthy as the evidence it leaves behind.

```text
No trace:
  "I think it worked."

Trace + ticket + artifact:
  "Here is exactly what happened."
```

The homework wants the second habit.

