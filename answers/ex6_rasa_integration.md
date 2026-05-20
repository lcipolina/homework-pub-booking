# Ex6 — Rasa structured half

## Your answer

The RasaStructuredHalf subclass overrides run() to POST a booking
intent to Rasa's REST webhook and interpret the response. Input
payload flows: loop half produces raw booking data → StructuredHalf
calls normalise_booking_payload (via validator.py) to produce a
Rasa-shaped message with canonical types → urllib POST to Rasa →
parse response for {action: committed} or {action: rejected} custom
slots.

For offline mode we spawn a stdlib http.server thread that mimics a
Rasa webhook. For real mode, `make ex6-auto` trains Rasa, starts the
action server and Rasa server, POSTs the booking, and tears both
processes down. In my real auto run, session `sess_b0e505eea2b8`
confirmed the Haymarket Tap booking with reference `BK-7D401E9E`.
The real response text was "Booking confirmed. Reference:
BK-7D401E9E.", followed by Rasa's default follow-up question.

Three design choices worth noting: (1) we raise ValidationFailed in
normalise_booking_payload and catch it in run() rather than letting
it propagate; the StructuredHalf contract demands a HalfResult. (2)
Network errors return success=False with SA_EXT_SERVICE_UNAVAILABLE
— the caller decides whether to retry. (3) The stable sender_id is a
hash of (venue+date+time) so the Rasa tracker is consistent across
retries within one session.

## Citations

- Real run `make ex6-auto`, session `sess_b0e505eea2b8` — Rasa confirmed reference `BK-7D401E9E`
- starter/rasa_half/validator.py — normalise_booking_payload + helpers
- starter/rasa_half/structured_half.py — RasaStructuredHalf.run + mock server
