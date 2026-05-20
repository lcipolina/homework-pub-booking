# Assignment Status Checklist

This file audits `ASSIGNMENT.md` against the current local repository state.

Legend:

- `[x]` Done and verified locally.
- `[~]` Implemented or partially verified, with a caveat.
- `[ ]` Not done / cannot be verified yet.

## Scenario

- [x] Research open pubs near Haymarket, check weather, work out catering cost, and produce a flyer.
  - Verified by `make ex5` and `make ex5-real`.
  - Real session: `sess_918142e47522`.
- [x] Confirm the booking under strict rules.
  - Verified by `make ex6` and `make ex6-auto`.
  - Real Rasa session: `sess_b0e505eea2b8`.
- [x] Handle the round trip if the pub manager declines.
  - Verified by `make ex7` and `make ex7-real`.
  - Real Rasa bridge session: `sess_3adc43074c06`.
- [x] Speak to the manager, simulated by an LLM persona, either in text mode or with real voice.
  - Text mode verified by `make ex8-text`.
  - Voice mode fallback verified by `make ex8-voice` without `SPEECHMATICS_KEY`.
  - Real voice verified by `make ex8-voice`, session `sess_813916a3547a`.

## Ex5 — Edinburgh Research Scenario

### Must Implement

- [x] `venue_search(near, party_size, budget_max_gbp)`.
  - Reads `venues.json`, filters open venues by area, seats, and budget.
- [x] `get_weather(city, date)`.
  - Reads `weather.json` and returns condition and temperature.
- [x] `calculate_cost(venue_id, party_size, duration_hours, catering_tier)`.
  - Reads `catering.json` and computes subtotal, service, total, and deposit.
- [x] `generate_flyer(event_details)`.
  - Writes `workspace/flyer.html`.
  - Note: README and current tests expect HTML (`flyer.html`), while `ASSIGNMENT.md` says markdown (`flyer.md`). Current repo behavior is HTML.
- [x] Read tools are parallel-safe.
- [x] `generate_flyer` is `parallel_safe=False`.
- [x] Every tool logs arguments and output into `_TOOL_CALL_LOG`.
- [x] Dataflow integrity check exists in `integrity.py`.
- [x] Dataflow check catches planted fabrication.
  - Verified by public test `test_verify_dataflow_catches_obvious_fabrication`.
- [x] Dataflow check does not false-positive on correct flyers.
  - Verified by `make ex5` / `make ex5-real`.
- [x] Runnable scenario in `run.py`.
- [x] Default mode uses `FakeLLMClient`.
- [x] `--real` uses OpenAI-compatible live provider.
  - Local config uses OpenRouter instead of Nebius due Nebius budget exhaustion.
- [x] Both modes end with `verify_dataflow`.
- [x] Session has successful planner/executor evidence.
  - Offline has planner/executor tickets.
  - Real Ex5 now uses a deterministic Ex5 planner plus live executor to avoid provider-specific planner drift.

### Grading Aspects

- [x] `make ex5` runs clean.
- [x] Flyer written to session workspace.
- [x] Fixture-backed tools work.
- [x] `generate_flyer` not parallel-safe.
- [x] Fabrication check catches fake facts.
- [x] Correct flyer passes integrity.
- [x] Tool calls are tracked for dataflow.

## Ex6 — Rasa Structured Half

### Must Implement

- [x] `RasaStructuredHalf` subclass routes booking data to Rasa via HTTP.
- [x] Rasa response is parsed back into `HalfResult`.
- [~] Rasa flows in `rasa_project/data/flows.yml`.
  - `confirm_booking` exists and is verified.
  - `resume_from_loop` and `request_research` are not separate flows in this repo; reverse handoff is handled in Python by Ex7's bridge.
- [x] `ActionValidateBooking` validates deposit <= £300.
- [x] `ActionValidateBooking` validates party size <= 8.
- [x] Rejection reason is returned through slots/responses.
- [x] `normalise_booking_payload` normalises booking data.
- [x] Date normalisation.
- [x] Currency normalisation.
- [x] Party size normalisation.
- [x] Time normalisation.
- [x] Venue ID canonicalisation.

### Grading Aspects

- [x] `make ex6` runs clean in mock mode.
- [x] `make ex6-auto` runs clean with real Rasa.
- [x] Valid booking commits.
- [x] Deposits over £300 are rejected by action logic.
- [x] Parties over 8 are rejected by action logic.
- [~] `resume_from_loop` as a named Rasa flow is absent; equivalent re-entry behavior is implemented at bridge level.
- [x] Validator normalises at least 3 required fields.

## Ex7 — Handoff Bridge

### Must Implement

- [x] Bridge logic in `bridge.py`.
- [x] Reads loop result / outgoing handoff payload.
- [x] Dispatches to Ex6 Rasa-backed structured half.
- [x] Writes/reconstructs reverse task when structured rejects.
- [x] End-to-end demo in `run.py`.
- [x] Starts with party of 12 near Haymarket.
- [x] Round 1 hands off to structured.
- [x] Structured rejects party-size violation.
- [x] Bridge returns to loop with rejection reason.
- [x] Round 2 proposes acceptable booking.
- [x] Structured approves.
- [x] Session reaches completed state.

### Grading Aspects

- [x] Forward handoff preserves context.
- [x] Reverse task preserves rejection reason.
- [x] Session completes within 3 rounds.
- [x] Stale handoff files are archived so visible IPC does not accumulate.
- [x] Trace contains `session.state_changed` events.
- [x] Max-rounds failure is reported if structured keeps rejecting.

## Ex8 — Voice Pipeline

### Must Implement

- [x] Manager persona exists in `manager_persona.py`.
- [x] Persona has gruff Edinburgh manager character.
- [x] Persona rules mention accepting <= 8 people and deposit <= £300.
- [x] Persona declines otherwise with a reason.
- [x] Text mode reads stdin and prints responses.
- [x] Voice mode function exists and is implemented.
- [x] Voice mode records microphone audio when keys/deps/mic are available.
- [x] Voice mode uses Speechmatics for STT.
- [x] Voice mode uses Rime TTS in current README/code.
  - Note: `ASSIGNMENT.md` says ElevenLabs, but README/current code say Rime. Current implementation follows README/code.
- [x] `voice.utterance_in` trace events emitted.
- [x] `voice.utterance_out` trace events emitted.
- [x] Missing `SPEECHMATICS_KEY` gracefully falls back to text mode.
  - Verified by `make ex8-voice`, session `sess_3cab2ade648e`.
- [x] `SPEECHMATICS_API_KEY` spelling is accepted as a compatibility alias.
  - The homework docs use `SPEECHMATICS_KEY`, but Speechmatics dashboards commonly describe API keys with the longer name.

### Grading Aspects

- [x] Text mode runs a full 3+ turn conversation.
  - Verified by `make ex8-text`, session `sess_aaf2d4c51905`.
- [x] Manager persona stays in character in tested valid-booking prompt.
- [x] Real voice end-to-end with STT/TTS is verified.
  - Verified by `make ex8-voice`, session `sess_813916a3547a`.
  - Trace contains `voice.utterance_in` and `voice.utterance_out` with `mode: voice`.
- [x] Every tested utterance is traced.
- [x] Missing-key graceful degradation verified.

### Should You Hear or Speak?

- In `make ex8-text`: no. You type and read text.
- In `make ex8-voice` without keys: no. It falls back to text mode.
- In `make ex8-voice` with `SPEECHMATICS_KEY` or `SPEECHMATICS_API_KEY`: yes, you should speak into the microphone and see transcribed text.
- With both `SPEECHMATICS_KEY` and `RIME_API_KEY`: yes, you should speak and hear the manager reply through speakers.

## Ex9 — Reflection

### Must Implement

- [x] `answers/ex9_reflection.md` exists.
- [x] Q1 answered.
- [x] Q2 answered.
- [x] Q3 answered.
- [x] Answers cite local run sessions.
  - Ex5 real: `sess_918142e47522`
  - Ex6 real Rasa: `sess_b0e505eea2b8`
  - Ex7 real Rasa: `sess_3adc43074c06`
  - Ex8 voice/text/fallback: `sess_813916a3547a`, `sess_aaf2d4c51905`, `sess_3cab2ade648e`
- [~] The answers are locally non-empty and pass mechanical checks.
  - Final reasoning quality is graded by CI/instructor LLM-as-judge.

### Grading Aspects

- [~] Specific session citations included.
  - Stronger final submission would cite exact trace/ticket paths or line numbers.
- [x] Each answer is substantive.
- [~] Grounding is plausible locally, but CI reasoning judge is authoritative.
- [~] Q3 currently names session directories as the primitive and cross-run data/reconstruction as failure mode.
  - Check final wording before submission if the judge expects exactly one named primitive and one narrowly stated failure.

## Integrity Requirements

- [x] Ex5 has dataflow integrity.
- [x] Ex7 has bridge integrity.
- [x] Ex8 has per-utterance trace audit.
- [x] No raw secrets are committed.
  - `.env` is gitignored.
- [x] No generated session artifacts are committed.

## Final Local Verification

- [x] `make verify` passes with OpenRouter.
- [x] `make test` passes: 27 public tests.
- [x] `make ci` passes.
- [x] `make check-submit` passes local non-reasoning: 46/46.

## Remaining Before Maximum Possible Score

- [x] Add `SPEECHMATICS_KEY` or `SPEECHMATICS_API_KEY` for real STT.
- [x] Add `RIME_API_KEY` for audible TTS.
- [x] Run `make ex8-voice` and complete a real spoken conversation.
- [x] Run a 3+ turn `make ex8-text` transcript if not doing real voice.
- [ ] Optionally strengthen Ex9 citations with exact trace paths or ticket files.
