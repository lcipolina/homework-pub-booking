# 01 — Sovereign Agent Architecture

## Why An Agent Framework Exists

An LLM API call is stateless:

```text
messages ──▶ model ──▶ response
```

That is not enough for a serious agent. A real agent run needs:

- durable state,
- tool execution,
- planning,
- retry handling,
- logs,
- artifacts,
- memory,
- boundaries between components.

The `sovereign-agent` framework provides these pieces so the homework can
focus on architecture rather than inventing all plumbing from scratch.

## Mental Model: Agent As A Small Operating System

Think of the agent framework as a tiny operating system for one task.

```text
┌──────────────────────────────────────────────────────────────┐
│ Agent Session                                                 │
│                                                              │
│  Task input                                                   │
│    │                                                         │
│    ▼                                                         │
│  Planner ──▶ Subgoals ──▶ Executor ──▶ Tool calls             │
│    │                         │                               │
│    │                         ▼                               │
│    │                    Artifacts/files                       │
│    │                         │                               │
│    ▼                         ▼                               │
│  Tickets                Trace events                          │
│                                                              │
│  Session directory stores the whole run.                      │
└──────────────────────────────────────────────────────────────┘
```

An operating system keeps processes, logs, files, and errors separate.
Likewise, a session keeps one agent run separate from another.

## Core Vocabulary

### Session

A session is one run of the agent.

It usually has:

- a session id, such as `sess_918142e47522`,
- a directory on disk,
- logs,
- tickets,
- workspace files,
- IPC files,
- trace events.

Why this matters:

> Without sessions, evidence from different runs gets mixed together.

In this homework, session directories let you prove things like:

- Ex5 produced the flyer in that run,
- Ex7 rejected round 1 and approved round 2,
- Ex8 recorded real voice events.

### Trace

A trace is an event log.

Example event types:

```text
planner.called
planner.produced_subgoals
executor.tool_called
session.state_changed
voice.utterance_in
voice.utterance_out
```

The trace answers:

- What happened?
- In what order?
- Which component did it?
- What payload did it carry?

Simplified trace:

```text
1. bridge.round_start       round=1, half=loop
2. planner.called           task="book for party of 12"
3. executor.tool_called     tool=venue_search
4. executor.tool_called     tool=handoff_to_structured
5. session.state_changed    loop -> structured
6. session.state_changed    structured -> loop, reason=party_too_large
7. bridge.round_start       round=2, half=loop
8. executor.tool_called     tool=handoff_to_structured, party_size=6
9. session.state_changed    structured -> complete
```

That is much stronger than a final sentence saying "it worked."

### Ticket

A ticket is a durable record of a unit of work.

Planner ticket:

```text
Task: book for party of 12 in Haymarket
Subgoals:
  sg_1 assigned_half="loop"
```

Executor ticket:

```text
Subgoal: sg_1
Tool calls:
  venue_search(...)
  handoff_to_structured(...)
Result:
  handoff requested
```

Tickets make the internal reasoning inspectable. They are particularly useful
for Ex9, because the reasoning judge wants citations grounded in actual run
artifacts.

### Workspace

The workspace is where an agent writes artifacts.

For Ex5, `generate_flyer` writes:

```text
workspace/flyer.html
```

The workspace is session-scoped, so a flyer from yesterday does not silently
count as today's flyer.

### IPC

IPC means inter-process communication.

In Ex7, the loop and structured halves communicate through handoff files:

```text
ipc/handoff_to_structured.json
```

This is a simple but powerful pattern: files become messages between
components.

## The Half Architecture

The homework uses "halves" because different components should own different
kinds of work.

```text
                  ┌─────────────────────┐
                  │ User task            │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Loop half            │
                  │ planner + executor   │
                  └──────────┬──────────┘
                             │ handoff
                             ▼
                  ┌─────────────────────┐
                  │ Structured half      │
                  │ Rasa + validators    │
                  └──────────┬──────────┘
                             │ approve/reject
                             ▼
                  ┌─────────────────────┐
                  │ Final state          │
                  └─────────────────────┘
```

The loop half is good at open-ended work:

- choose tools,
- research,
- try alternatives,
- summarize.

The structured half is good at controlled decisions:

- validate fields,
- enforce limits,
- return typed approval/rejection.

## Planner And Executor

The planner decides what needs doing.

The executor does it.

```text
Task:
  "Find a pub and make a flyer."

Planner:
  "Subgoal 1: search venues."
  "Subgoal 2: get weather."
  "Subgoal 3: calculate cost."
  "Subgoal 4: generate flyer."

Executor:
  Calls venue_search
  Calls get_weather
  Calls calculate_cost
  Calls generate_flyer
```

This split matters because a planner can be wrong. In our implementation,
Ex5 real mode uses a deterministic planner to avoid provider drift while still
using a live LLM executor.

That is an important engineering compromise:

> Use an LLM where flexibility helps; use deterministic logic where the
> assignment requires a stable route.

## Why Logging Is Not Optional

Agent failures are often not obvious from the final output.

Example:

```text
Final:
  "The booking is confirmed."

Possible hidden histories:
  A. Rasa approved it.
  B. Rasa rejected it, but the LLM ignored the rejection.
  C. Rasa never ran.
  D. The LLM invented the confirmation.
```

Only traces and tickets tell the difference.

That is why the homework keeps asking for:

- session ids,
- trace evidence,
- tool-call logs,
- citations in Ex9.

## Why This Is Called A Production Pattern

Production agent systems need:

- observability,
- reproducibility,
- isolation,
- error recovery,
- policy boundaries.

This homework is small, but the same pattern scales:

```text
Customer support agent:
  LLM drafts response.
  Policy engine checks refund eligibility.
  CRM tool fetches account state.
  Trace records all actions.

Medical intake agent:
  LLM asks questions.
  Structured triage rules classify urgency.
  Logs preserve evidence.

Travel booking agent:
  LLM searches options.
  Pricing API provides facts.
  Policy engine enforces budget.
```

The lesson is not "pubs." The lesson is controlled autonomy.

