# 09 — Output Review Guide

This guide answers two practical questions:

1. Where did we write conclusions?
2. Where do I inspect model outputs and consistency?

## Where The Conclusions Are

The homework conclusions live in:

```text
answers/
```

The most important file is:

```text
answers/ex9_reflection.md
```

That is the CI-graded reasoning section. It explains what happened in the
real runs and cites evidence from sessions, traces, and tickets.

The other answer files are shorter exercise-level conclusions:

```text
answers/ex5_loop_scenario.md
answers/ex6_rasa_integration.md
answers/ex7_handoff_bridge.md
answers/ex8_voice_pipeline.md
answers/ex9_reflection.md
```

Mental model:

```text
answers/ex5-8 = what each exercise did
answers/ex9   = deeper conclusions and evidence-based reflection
```

## Where The Outputs Are

Most runtime outputs are not stored inside the Git repo. They are in session
directories under:

```text
/Users/lucia/Library/Application Support/sovereign-agent/
```

Important sessions from this homework:

```text
Ex5 real loop/flyer:
  examples/ex5-edinburgh-research/sess_918142e47522

Ex6 real Rasa:
  examples/ex6-rasa-half/sess_b0e505eea2b8

Ex7 real handoff:
  examples/ex7-handoff-bridge/sess_3adc43074c06

Ex8 real voice:
  homework/ex8/sess_813916a3547a

Ex8 text comparison:
  homework/ex8/sess_aaf2d4c51905
```

## What To Open For Each Exercise

### Ex5

Open:

```text
.../sess_918142e47522/workspace/flyer.html
.../sess_918142e47522/logs/trace.jsonl
.../sess_918142e47522/SESSION.md
```

Look for:

- tool calls to `venue_search`, `get_weather`, `calculate_cost`,
  `generate_flyer`,
- the generated flyer,
- dataflow verification success.

Command:

```bash
make narrate SESSION=sess_918142e47522
```

### Ex6

Open:

```text
.../sess_b0e505eea2b8/logs/trace.jsonl
.../sess_b0e505eea2b8/SESSION.md
```

Look for:

- normalized booking payload,
- Rasa approval,
- booking reference.

### Ex7

Open:

```text
.../sess_3adc43074c06/logs/trace.jsonl
.../sess_3adc43074c06/logs/tickets/
```

Look for:

- round 1 loop to structured,
- rejection reason `party_too_large`,
- reverse handoff back to loop,
- round 2 smaller booking,
- structured to complete.

Useful search:

```bash
rg "party_too_large|session.state_changed|handoff_to_structured" \
  "/Users/lucia/Library/Application Support/sovereign-agent/examples/ex7-handoff-bridge/sess_3adc43074c06/logs"
```

### Ex8

Open:

```text
.../sess_813916a3547a/logs/trace.jsonl
.../sess_813916a3547a/workspace/turn_0_input.wav
```

Look for:

- `voice.utterance_in` with `mode: "voice"`,
- `voice.utterance_out` with `mode: "voice"`,
- the transcribed user request,
- the manager's policy-grounded rejection.

The rejection for 160 guests is correct because the manager persona only
accepts parties of 8 or fewer.

## How To Check Consistency

Use this checklist:

```text
1. Does the final answer cite a real session id?
2. Does the trace contain the event described in the answer?
3. Does the final artifact use facts returned by tools?
4. Does Rasa's decision match the policy?
5. If rejected, does Ex7 show a retry?
6. Does Ex8 voice trace say mode="voice"?
```

Consistency examples:

```text
Ex5:
  Flyer money/weather facts should match tool outputs.

Ex7:
  Ex9 says party_too_large; trace should contain party_too_large.

Ex8:
  Answer says real voice; trace should contain mode="voice".
```

## Commands For Review

```bash
make check-submit
make ci
make narrate SESSION=sess_918142e47522
make narrate SESSION=sess_3adc43074c06
```

For raw trace inspection:

```bash
tail -n 20 "<session>/logs/trace.jsonl"
```

For ticket inspection:

```bash
find "<session>/logs/tickets" -maxdepth 2 -type f | sort
```

## What Good Performance Looks Like Here

For this homework, "good performance" does not mean the model always gets a
booking accepted. It means:

- tools are called correctly,
- unsupported facts are caught,
- policy rejections are respected,
- handoff recovery happens,
- voice input/output is traced,
- written conclusions cite evidence.

The model rejecting 160 guests in Ex8 is therefore good behavior. It shows
the persona followed the policy instead of trying to please the user.

