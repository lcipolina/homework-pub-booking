# 00 — What This Homework Is About

## Overview

Week 5 asks you to build an AI agent that books an Edinburgh pub. The story
sounds simple: find a pub, confirm the booking, and talk to the manager. The
architecture underneath is the real lesson.

You are building a small production-style agent system with these parts:

- **Ex5:** a loop half researches venues, checks weather, calculates cost,
  and writes an HTML flyer.
- **Ex6:** a Rasa CALM structured half validates booking policy.
- **Ex7:** a handoff bridge sends rejected bookings back to the loop half.
- **Ex8:** a voice pipeline connects speech-to-text, manager persona, and
  text-to-speech.
- **Ex9:** reflection answers grounded in your logs and traces.

## Product Goal

The system should help with a concrete operations task:

> “Book a suitable pub in Edinburgh for a group, under policy constraints,
> and produce auditable evidence of how the decision was made.”

That means the agent must not merely sound plausible. It must:

- call tools for facts,
- compute costs deterministically,
- obey party-size and deposit policy,
- produce traceable artifacts,
- recover when a structured validator rejects a proposal.

## Big Architecture Idea

The homework separates agent work into two styles.

### Loop Half

The loop half is exploratory. It is good at:

- searching,
- comparing,
- deciding which tool to call next,
- drafting outputs like a flyer.

It is risky for:

- policy decisions,
- money constraints,
- final booking confirmation.

### Structured Half

The structured half is controlled. It is good at:

- validating fields,
- applying business rules,
- producing predictable accept/reject outcomes.

It is less flexible for:

- open-ended research,
- discovering alternatives.

### Bridge

The bridge coordinates the two:

1. loop proposes a booking,
2. structured validates it,
3. if accepted, complete,
4. if rejected, send the reason back to the loop and try again.

This is a pattern you will see in real systems: autonomy where ambiguity is
high, structure where correctness matters.

## What You Should Learn

You should be able to explain:

- why tool calls are evidence,
- why deterministic code should handle arithmetic and policy,
- why traces matter for debugging,
- why “real mode” can fail even when offline tests pass,
- why a voice interface is a transport layer, not a different brain.

## Exam Checklist

Be ready to answer:

1. What is the difference between a loop half and a structured half?
2. Why does Ex5 need a dataflow integrity check?
3. Why does Ex7 need reverse handoff?
4. What does Rasa add that a plain LLM loop does not?
5. What evidence would prove your agent really did the work?
