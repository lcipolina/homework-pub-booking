# 02 — Loop Half, Tools, and Dataflow Integrity

## Overview

Ex5 is the loop-half exercise. The agent researches an Edinburgh pub and writes
an HTML flyer. The model is allowed to decide tool calls, but the tools provide
the ground truth.

## The Four Ex5 Tools

### `venue_search`

Reads `sample_data/venues.json` and filters by:

- area,
- open status,
- available seats,
- hire fee plus minimum spend.

This is not an internet search. It is a deterministic fixture-backed tool.

### `get_weather`

Reads `sample_data/weather.json` and returns scripted weather facts:

- condition,
- temperature,
- rain,
- wind.

### `calculate_cost`

Reads `sample_data/catering.json` and computes:

- subtotal,
- service charge,
- total,
- deposit.

The model should not do arithmetic itself. Money logic belongs in code.

### `generate_flyer`

Writes `workspace/flyer.html`.

Important: this tool is `parallel_safe=False` because it writes a file. The
read-only tools can run in parallel, but a writer must not be batched blindly.

## Tool Contract

A strong tool has:

- typed arguments,
- stable output keys,
- structured error behavior,
- no hidden side effects unless clearly declared,
- a short summary for trace readability.

The model sees the tool schema and summaries. If the tool contract is vague,
the model improvises.

## Dataflow Integrity

The flyer is human-facing, so it is a hallucination risk. An LLM could write:

```text
Total: £9999
```

even if no tool returned that number.

`verify_dataflow()` checks concrete facts in the flyer against logged tool
outputs. It focuses on facts like:

- money,
- temperature,
- weather condition.

If a fact appears in the flyer but not in a tool result, the flyer is not
grounded.

## Real-Mode Failure We Saw

With OpenRouter, the first Ex5 real run failed because the planner invented a
different plan:

- it searched for party size `10`,
- it assigned flyer work to `structured`,
- it never wrote `flyer.html`.

The fix was architectural: Ex5 has a deterministic required sequence, so the
planner can be fixed while the executor still uses the live LLM for tool calls.

## Key Lesson

Autonomy should be bounded by the assignment contract. A live model is useful
for action selection, but deterministic scaffolding is safer when the task has
explicit required steps.

## Exam Checklist

You should be able to answer:

1. Which Ex5 tools are read-only and why?
2. Why is `generate_flyer` not parallel-safe?
3. What does dataflow integrity catch?
4. Why should cost calculation be deterministic?
5. What is the difference between a tool failure and an LLM planning failure?
