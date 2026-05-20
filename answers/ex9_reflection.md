# Ex9 — Reflection

## Q1 — Planner handoff decision

### Your answer

In my Ex7 real run (session `sess_3adc43074c06`), the first planner
ticket (`logs/tickets/tk_5b8dcd34/raw_output.json`) produced subgoal
`sg_1` with `assigned_half: "loop"` and description "retry with larger
venue after rejection." The loop executor then called
`handoff_to_structured` with context "party of 12 near Haymarket" and
booking data containing `party_size: "12"`.

The decisive signal was the bridge trace, not the natural-language
summary. `logs/trace.jsonl` records the transition from loop to
structured in round 1, then immediately records structured returning to
loop with `rejection_reason: "sorry, we can't accept this booking.
reason: party_too_large"`. That state transition forced a second loop
round instead of allowing the first proposal to count as complete.

Round 2 shows the handoff decision working as a control mechanism. The
next executor ticket proposed a smaller booking: `party_size: "6"` at
The Royal Oak, with reason "retry after reverse handoff — scaled down to
fit policy." The trace then records structured moving to `complete`.

The lesson is that planner assignment alone is advisory; the bridge must
enforce typed transitions. Here, `assigned_half: "loop"` caused research,
but only the explicit `handoff_to_structured` call and the structured
half's rejection/approval states made the booking outcome auditable.

### Citation

- Real run `make ex7-real`, session `sess_3adc43074c06`
- `logs/tickets/tk_5b8dcd34/raw_output.json` — subgoal `sg_1`, `assigned_half: "loop"`
- `logs/trace.jsonl` — loop→structured, structured→loop rejection, structured→complete
- starter/handoff_bridge/bridge.py — `HandoffBridge.run`

---

## Q2 — Dataflow integrity catch

### Your answer

The Ex5 integrity check is valuable because the flyer is not trusted
just because it is well-formatted HTML. In my real `make ex5-real` run
(session sess_918142e47522), the scenario produced `flyer.html` and
`verify_dataflow` reported `dataflow OK: verified 4 fact(s) against
tool outputs`. Those facts included the money values and the weather
condition/temperature.

The test I care about is the opposite path: when the flyer says
`£9999` but no tool returned that value, `verify_dataflow` flags it.
That matters because `£9999` is not semantically impossible; it is
just unsupported by this session's tools. A plausibility check would
miss smaller fabrications like a believable but wrong deposit.

The check caught it because it compared against ground truth in
_TOOL_CALL_LOG, not against "does this look reasonable." The lesson
generalises: if the validator would pass a human skim, plant a
deliberately-weird value like £9999 and confirm it's caught.

### Citation

- Real run `make ex5-real`, session `sess_918142e47522`
- tests/public/test_ex5_scaffold.py — fabrication check for `£9999`

---

## Q3 — Removing one framework primitive

### Your answer

I'd keep exactly one primitive: session directories. They are the
boundary that makes every run inspectable as its own object. In this
homework, each successful run printed a concrete session id:
`sess_918142e47522` for Ex5, `sess_b0e505eea2b8` for Ex6,
`sess_3adc43074c06` for Ex7, and `sess_813916a3547a` for Ex8 voice.

The one failure mode I would expect without session directories is
cross-run evidence contamination. A rejected booking from one attempt
and an approved booking from a later attempt could share logs,
handoff files, or trace events. Then the grader, or I, could not prove
which tool output caused which final answer.

The Ex7 run shows why this matters. Its session directory contains the
round-1 rejection for `party_too_large` and the round-2 completion after
the party was reduced to six. Keeping those artifacts together prevents
the most dangerous debugging mistake: explaining the final success using
evidence from a different run.

### Citation

- Real/text runs: `sess_918142e47522`, `sess_b0e505eea2b8`, `sess_3adc43074c06`, `sess_813916a3547a`, `sess_aaf2d4c51905`
- .gitignore — documents why persistent `sessions/` artifacts are not committed
