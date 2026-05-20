# 03 — Rasa CALM Structured Half

## Overview

Ex6 introduces the structured half. It uses Rasa Pro CALM to validate a booking
against policy rules.

The structured half is not supposed to research. It is supposed to decide:

> “Can this proposed booking be confirmed under policy?”

## Why Rasa?

A plain LLM loop can be creative, but booking confirmation needs predictability.
Rasa provides:

- explicit flows,
- slots,
- custom actions,
- a REST webhook,
- auditable accept/reject behavior.

## Main Files

- `rasa_project/config.yml` — Rasa pipeline and policy config.
- `rasa_project/domain.yml` — slots, actions, responses.
- `rasa_project/data/flows.yml` — CALM flow definition.
- `rasa_project/actions/actions.py` — deterministic booking validation.
- `starter/rasa_half/validator.py` — converts raw booking data to Rasa shape.
- `starter/rasa_half/structured_half.py` — POSTs to Rasa and parses response.

## Payload Normalisation

The loop half may send loose data:

```python
{
    "venue_id": "Haymarket Tap",
    "date": "25th April 2026",
    "time": "7:30pm",
    "party_size": "6",
    "deposit": "£200",
}
```

Rasa should receive canonical data:

```python
{
    "venue_id": "haymarket_tap",
    "date": "2026-04-25",
    "time": "19:30",
    "party_size": 6,
    "deposit_gbp": 200,
}
```

That is the job of `normalise_booking_payload()`.

## Custom Action

`ActionValidateBooking` reads booking data from message metadata and applies
rules:

- party size greater than 8 means reject,
- deposit greater than £300 means reject,
- missing required fields mean reject,
- otherwise confirm and set a booking reference.

This is deterministic Python. That is exactly where business rules belong.

## Rasa Real Mode

Real Rasa mode needs:

1. `rasa train`,
2. action server on port `5055`,
3. Rasa server on port `5005`,
4. the Python scenario POSTing to `/webhooks/rest/webhook`.

`make ex6-auto` performs those steps in one command.

## Provider Configuration

Rasa has its own LLM and embedding config in:

```text
rasa_project/endpoints.yml
```

That file must match your provider. In our current local setup, it uses
OpenRouter for both command generation and embeddings.

## Exam Checklist

You should be able to explain:

1. Why Rasa is used for booking confirmation.
2. What slot values are needed for confirmation.
3. Why metadata is used to pass booking data.
4. What `ActionValidateBooking` enforces.
5. Why Rasa needs embeddings during training.
