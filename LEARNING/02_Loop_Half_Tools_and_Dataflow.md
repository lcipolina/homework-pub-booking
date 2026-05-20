# 02 — Loop Half, Tools, And Dataflow Integrity

## What Is The Loop Half?

The loop half is the agent part that behaves most like a classic LLM agent:

```text
observe task ──▶ think/plan ──▶ call tool ──▶ observe result ──▶ continue
```

It is called a loop because the agent repeats:

1. choose next action,
2. call a tool or produce an answer,
3. inspect what happened,
4. decide whether more work is needed.

In Ex5, the loop half must:

- search venues,
- get weather,
- calculate catering cost,
- generate a flyer,
- verify the flyer did not fabricate important facts.

## The Ex5 Pipeline

```text
┌──────────────┐
│ User task    │
│ "pub flyer"  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ venue_search │  reads fixtures/venues.json
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ get_weather  │  reads fixtures/weather.json
└──────┬───────┘
       │
       ▼
┌────────────────┐
│ calculate_cost │  reads fixtures/catering.json
└──────┬─────────┘
       │
       ▼
┌────────────────┐
│ generate_flyer │  writes workspace/flyer.html
└──────┬─────────┘
       │
       ▼
┌─────────────────┐
│ verify_dataflow │  checks flyer claims against tool logs
└─────────────────┘
```

## What Is A Tool?

An agent tool is a function the LLM can call.

Normal Python function:

```text
input ──▶ function ──▶ output
```

Agent tool:

```text
LLM intention ──▶ structured arguments ──▶ function ──▶ structured result
```

For example:

```python
venue_search(near="Haymarket", party_size=160, budget_max_gbp=2000)
```

The LLM does not directly inspect the whole `venues.json` file. It asks the
tool a constrained question. The tool returns grounded data.

## Why Tools Need Contracts

A tool contract says:

- what arguments are accepted,
- what types they should have,
- what the tool returns,
- whether the tool is safe to run in parallel,
- what errors look like.

Without a contract, the LLM might call:

```python
venue_search("somewhere nice", "a lot", "cheap")
```

That is human-readable, but bad software input.

A good contract forces the messy user request into structured data:

```json
{
  "near": "Haymarket",
  "party_size": 160,
  "budget_max_gbp": 2000
}
```

## The Four Ex5 Tools

### 1. `venue_search`

Purpose:

- find venues near an area,
- filter by capacity,
- filter by budget,
- return structured venue facts.

Key concept:

> This is retrieval over a trusted local fixture, not free-form web search.

Why that matters:

- deterministic tests,
- no live network failures,
- reproducible grading,
- facts can be checked later.

### 2. `get_weather`

Purpose:

- retrieve weather for a city/date,
- provide condition and temperature for the flyer.

Key concept:

> The weather fact must come from a tool if the flyer mentions it.

If the flyer says "sunny, 22C" but the tool returned "rain, 11C," integrity
should fail.

### 3. `calculate_cost`

Purpose:

- compute catering totals,
- calculate deposit,
- return money values.

Key concept:

> Arithmetic should be code, not model imagination.

LLMs can do arithmetic, but they are not reliable calculators. Costing belongs
in deterministic code.

### 4. `generate_flyer`

Purpose:

- create the final HTML artifact.

Important detail:

```text
generate_flyer is not parallel-safe.
```

Why?

It writes to a fixed output file. If two calls run at the same time, they could
overwrite or interleave output.

```text
Parallel-safe:
  venue_search, get_weather, calculate_cost

Not parallel-safe:
  generate_flyer
```

## Parallel Safety Infographic

```text
Read-only tools:

  Tool A reads fixtures ──▶ result
  Tool B reads fixtures ──▶ result
  Tool C reads fixtures ──▶ result

  Safe to run together because nobody changes shared state.

Write tool:

  generate_flyer ──▶ workspace/flyer.html
  generate_flyer ──▶ workspace/flyer.html

  Dangerous together because both write the same artifact.
```

## What Is Dataflow?

Dataflow is the path facts take through a system.

Example:

```text
venues.json
  │
  ▼
venue_search output
  │
  ▼
tool call log
  │
  ▼
flyer.html
  │
  ▼
verify_dataflow
```

If a fact appears in the flyer, we should be able to trace it backward.

```text
Flyer says:
  "Deposit: £320"

Question:
  Did any tool return £320?

If yes:
  grounded fact

If no:
  possible fabrication
```

## What Is Fabrication?

Fabrication is when the model outputs a fact not supported by evidence.

In ordinary language, this is "making something up."

In this homework, examples include:

- inventing a venue,
- inventing a cost,
- changing the deposit,
- inventing weather,
- claiming capacity that the tool did not return.

The dangerous part is that fabricated facts can be plausible.

```text
Obviously fake:
  Deposit: £9999

Plausible but still fake:
  Deposit: £250
```

The integrity checker should care about support, not plausibility.

## Tool Call Log

The implementation records calls in `_TOOL_CALL_LOG`.

Conceptually:

```json
[
  {
    "tool": "venue_search",
    "arguments": {"near": "Haymarket", "party_size": 160},
    "output": {"venues": [...]}
  },
  {
    "tool": "calculate_cost",
    "arguments": {"venue_id": "haymarket_tap"},
    "output": {"deposit_gbp": 320}
  }
]
```

The log becomes the source of truth for integrity.

## Integrity Check Mental Model

```text
             ┌─────────────────────┐
             │ Tool outputs         │
             │ trusted facts        │
             └──────────┬──────────┘
                        │
                        ▼
┌──────────────┐   compare facts   ┌───────────────────┐
│ flyer.html   │ ────────────────▶ │ verify_dataflow   │
│ final claims │                   │ pass/fail report  │
└──────────────┘                   └───────────────────┘
```

This is not perfect general truth verification. It is scoped verification:

> For selected important facts, check whether the final output is supported by
> the tools used in this run.

## Why The Grader Plants `£9999`

The grader may modify the flyer to include a fake value like `£9999`.

That tests whether your integrity checker is real.

Bad checker:

```python
return IntegrityResult(ok=True)
```

This passes the happy path but fails the planted-fabrication test.

Good checker:

```text
Does £9999 appear in tool outputs?
  no -> fail
```

## Common Mistakes

### Mistake 1: Letting The LLM Calculate Cost

Wrong:

```text
LLM: "For 160 people, I estimate £1,200."
```

Right:

```text
calculate_cost(...) returns exact subtotal/service/deposit.
```

### Mistake 2: Generating A Nice Flyer Without Evidence

Wrong:

```text
The flyer looks professional, therefore done.
```

Right:

```text
The flyer looks professional and every checked fact appears in tool outputs.
```

### Mistake 3: Treating Tool Errors As Text Decoration

If a tool fails, the loop should recover or stop honestly. It should not hide
the failure inside a polished artifact.

## Repo Files To Study

- `starter/edinburgh_research/tools.py`
  - tool implementations,
  - logging,
  - input validation,
  - flyer generation.

- `starter/edinburgh_research/integrity.py`
  - integrity result shape,
  - dataflow verification.

- `starter/edinburgh_research/run.py`
  - scenario orchestration,
  - real vs fake LLM modes.

- `tests/public/test_ex5_scaffold.py`
  - what the public tests expect.

## The Big Lesson

The loop half gives the agent flexibility. Tools give it grounded facts.
Dataflow integrity checks whether the flexible output stayed attached to those
facts.

That is one of the most important patterns in applied LLM systems:

```text
LLM freedom + deterministic tools + evidence checks = controlled usefulness
```

