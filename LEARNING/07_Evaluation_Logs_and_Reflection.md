# 07 — Evaluation, Logs, and Reflection

## Overview

The homework grade has three layers:

- mechanical checks,
- behavioral checks,
- reasoning/reflection checks.

Local grading can verify the first two. CI or instructor review may evaluate
the reasoning layer.

## Mechanical Checks

Mechanical checks ask:

- do files exist?
- does code import?
- does ruff pass?
- do public tests pass?
- are answers non-empty?

This is necessary but not sufficient.

## Behavioral Checks

Behavioral checks ask:

- does Ex5 run end-to-end?
- does Ex6 structured half confirm/reject correctly?
- does Ex7 bridge complete a round trip?
- is Ex8 voice mode implemented?

Behavioral checks are closer to the real homework goals.

## Reasoning Checks

Reasoning checks ask:

- can you explain what happened in your run?
- do your claims match trace evidence?
- do you understand the failure modes?
- can you justify architecture decisions?

This is where generic answers lose marks.

## Evidence Sources

Use:

- `logs/trace.jsonl`,
- `logs/tickets/*/raw_output.json`,
- command output,
- generated `workspace/flyer.html`,
- `make narrate SESSION=...`.

Good answer style:

```text
In session sess_..., the agent first called venue_search with
near='Haymarket' and party_size=6. The tool returned 1 result.
It then called get_weather and calculate_cost before generate_flyer.
```

Weak answer style:

```text
The agent did a good job and used tools properly.
```

## Real-Mode Evidence From Our Run

Useful sessions:

- Ex5 real success: `sess_918142e47522`
- Ex6 real Rasa auto success: `sess_b0e505eea2b8`
- Ex7 real Rasa bridge success: `sess_3adc43074c06`

These show:

- OpenRouter provider works for Ex5,
- Rasa can train and run with OpenRouter config,
- the bridge completes a two-round policy validation path.

## Exam Checklist

You should be able to answer:

1. What does `make check-submit` measure?
2. Why is a passing public test suite not the same as full correctness?
3. How do you cite trace evidence?
4. What is a behavioral failure?
5. Why should reflection answers mention exact session behavior?
