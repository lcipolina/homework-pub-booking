# 03 — Rasa CALM And The Structured Half

## Why Add Rasa When We Already Have An LLM?

An LLM is good at language. It is not automatically good at enforcing policy.

Booking policy in this homework is simple:

```text
Accept only if:
  party_size <= 8
  deposit <= £300

Reject otherwise with a reason.
```

You could prompt an LLM:

```text
"Please reject deposits above £300."
```

But prompting is not enforcement. The model might forget, misread, or be
persuaded by conversational context.

Rasa gives the system a structured dialog/policy layer. It is where the
business rule lives.

## The Core Split

```text
LLM loop half:
  "I found a possible venue. Here is the booking request."

Rasa structured half:
  "Does this request satisfy the booking policy?"

Bridge:
  "If yes, complete. If no, send reason back to loop."
```

The structured half is not supposed to be creative. Its job is to be reliable.

## What Is Rasa?

Rasa is a conversational AI framework. It can:

- define flows,
- track slots,
- call custom actions,
- validate data,
- choose responses,
- manage dialog state.

In this homework, Rasa acts like a strict booking manager.

```text
Incoming booking payload
        │
        ▼
Rasa flow / action
        │
        ▼
Validation rules
        │
        ├── approved -> booking reference
        │
        └── rejected -> reason
```

## What Is CALM?

CALM is Rasa's newer approach to building conversational assistants with LLMs
and structured flows.

For this homework, do not get lost in product terminology. The practical idea
is:

> Rasa is responsible for the controlled dialog and policy decision, while
> the loop half remains responsible for open-ended research.

## Structured Data vs Natural Language

Natural language:

```text
"Can we book Haymarket Tap tonight for a dozen people? Deposit maybe 500."
```

Structured booking payload:

```json
{
  "action": "confirm_booking",
  "venue_id": "Haymarket Tap",
  "date": "2026-04-25",
  "time": "19:30",
  "party_size": "12",
  "deposit": "£500"
}
```

Rasa and validators work better on the structured version.

## The Validation Pipeline

Before Rasa can enforce policy, the payload must be normalized.

```text
Raw payload
  party_size: "six"
  deposit: "£200"
  time: "7:30pm"
        │
        ▼
normalise_booking_payload
        │
        ▼
Normalized payload
  party_size: 6
  deposit_gbp: 200
  time: "19:30"
        │
        ▼
Rasa action validates policy
```

Normalization is not optional. It converts messy real-world inputs into types
that policy code can reason about.

## Slots

A slot is a piece of dialog state.

In a booking flow, slots might include:

```text
venue_id
date
time
party_size
deposit_gbp
contact_name
rejection_reason
booking_reference
```

Slots are the structured memory of the conversation.

```text
User/loop says: "six people"
        │
        ▼
slot party_size = 6
```

## Custom Actions

A custom action is Python code Rasa calls during a conversation.

In this homework, the important action is conceptually:

```text
ActionValidateBooking
```

It checks:

```text
if party_size > 8:
    reject party_too_large

elif deposit_gbp > 300:
    reject deposit_too_high

else:
    approve booking
```

This is the heart of the structured half.

## Why Validation Should Be Code

Policy enforcement is a bad place for fuzzy language.

Bad:

```text
LLM: "That deposit seems a little high, but maybe acceptable."
```

Good:

```python
deposit_gbp <= 300
```

Business rules should be:

- explicit,
- tested,
- auditable,
- hard to negotiate away.

## Ex6 Flow

```text
Loop or test payload
        │
        ▼
normalise_booking_payload
        │
        ▼
RasaStructuredHalf.run
        │
        ▼
POST to Rasa HTTP endpoint
        │
        ▼
Rasa flow/action
        │
        ▼
HalfResult
  status: approved/rejected
  output: booking_reference or rejection_reason
```

## What Is A `HalfResult`?

`HalfResult` is the standard shape used to report what a half did.

Conceptually:

```json
{
  "status": "completed",
  "output": {
    "approved": true,
    "booking_reference": "BK-..."
  },
  "next_action": "complete"
}
```

or:

```json
{
  "status": "rejected",
  "output": {
    "approved": false,
    "rejection_reason": "party_too_large"
  },
  "next_action": "escalate"
}
```

The exact object is Python, but the mental model is JSON-like:

> A half returns typed status, not just prose.

## Why Three Terminals?

Rasa in real mode needs multiple processes:

```text
Terminal 1:
  make rasa-actions
  Runs custom Python actions.

Terminal 2:
  make rasa-serve
  Runs the Rasa server.

Terminal 3:
  make ex6-real or ex7-real
  Runs your agent scenario.
```

The reason is architectural: Rasa server and Rasa actions are separate
services.

```text
Agent ──HTTP──▶ Rasa server ──HTTP──▶ Action server
```

Our implementation also added auto lifecycle support, so some commands can
start the required services for you.

## Common Rasa Failure Modes

### 1. Action Server Not Restarted

If you edit:

```text
rasa_project/actions/
```

you must restart the action server. Otherwise, Rasa keeps using old Python
code in memory.

### 2. Wrong Rasa Project Path

If the code points outside the repo, training/server startup fails. We fixed
the project path so it resolves to:

```text
homework-pub-booking/rasa_project
```

### 3. Provider Configuration Problems

Rasa may need an LLM/embedding provider. The repo originally expected Nebius.
We configured OpenRouter because the available Nebius key had exhausted
budget.

### 4. Silent Normalization Defaults

Bad validators silently default:

```text
"abc 500" -> 0
```

Good validators reject malformed input. This matters because policy checks are
only as good as the normalized values.

## Repo Files To Study

- `starter/rasa_half/validator.py`
  - parsing currency,
  - parsing time,
  - party-size validation,
  - payload normalization.

- `starter/rasa_half/structured_half.py`
  - sends payloads to Rasa,
  - turns responses into `HalfResult`.

- `rasa_project/actions/actions.py`
  - custom action logic.

- `rasa_project/data/flows.yml`
  - Rasa flow definition.

- `rasa_project/endpoints.yml`
  - provider configuration.

## The Big Lesson

Use LLMs for flexible language and exploration. Use structured systems for
policy decisions.

```text
LLM:
  "What could work?"

Rasa:
  "Is this allowed?"
```

That separation is what keeps the agent from sounding confident while making
an invalid booking.

