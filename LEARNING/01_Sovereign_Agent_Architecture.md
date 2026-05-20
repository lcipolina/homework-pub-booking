# 01 — Sovereign Agent Architecture

## Overview

The homework uses `sovereign-agent`, an agent framework built around
auditable sessions, tool execution, and communication between “halves.”

The important concepts are:

- **Session:** one run of an agent task.
- **Workspace:** files produced by the session, such as `flyer.html`.
- **Logs:** trace events and ticket outputs.
- **Ticket:** an auditable unit of work such as planner or executor run.
- **Loop half:** the autonomous tool-using side.
- **Structured half:** the controlled workflow side.
- **Handoff:** the data object passed between halves.

## Session Directories

Each real run creates a session directory, for example:

```text
~/Library/Application Support/sovereign-agent/examples/ex5-edinburgh-research/sess_...
```

Inside it you usually find:

- `session.json` — session metadata,
- `logs/trace.jsonl` — chronological events,
- `logs/tickets/` — raw planner/executor outputs,
- `workspace/` — generated files such as `flyer.html`,
- IPC/handoff files when halves communicate.

This matters because your final answers should describe what actually happened
in your run, not what you hoped happened.

## Tickets

A ticket is a traceable operation. In Ex5 you may see tickets like:

- `planner.plan`,
- `executor.run_subgoal/sg_1`,
- `executor.run_subgoal/sg_2`.

The raw ticket output tells you what the model planned or executed. If the
agent claims it did something, the ticket and trace should prove it.

## Trace Events

The trace is the source of truth for behavior. Typical events include:

- `planner.called`,
- `planner.produced_subgoals`,
- `executor.tool_called`,
- `session.state_changed`,
- `bridge.round_start`,
- `voice.utterance_in`,
- `voice.utterance_out`.

When debugging, always ask:

1. What tool was called?
2. With what arguments?
3. Did it succeed?
4. What did the model do after seeing the result?

## Why This Architecture Exists

LLM outputs are probabilistic. Production systems need:

- reproducible records,
- bounded side effects,
- separation between planning and policy,
- recovery paths when something fails.

The framework’s architecture makes those concerns explicit.

## Code Locations

- Ex5 runner: `starter/edinburgh_research/run.py`
- Ex5 tools: `starter/edinburgh_research/tools.py`
- Rasa half: `starter/rasa_half/structured_half.py`
- Handoff bridge: `starter/handoff_bridge/bridge.py`
- Voice pipeline: `starter/voice_pipeline/voice_loop.py`

## Exam Checklist

You should be able to explain:

- why session directories are better than ad-hoc print logs,
- what a ticket represents,
- why traces are needed for grading and debugging,
- how a handoff differs from a normal tool call.
