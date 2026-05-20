# Appendix — Commands and Troubleshooting

## Setup

```bash
make setup
make verify
```

If using OpenRouter locally, `.env` should include:

```env
OPENROUTER_API_KEY=...
SOVEREIGN_AGENT_LLM_BASE_URL=https://openrouter.ai/api/v1
SOVEREIGN_AGENT_LLM_API_KEY_ENV=OPENROUTER_API_KEY
SOVEREIGN_AGENT_LLM_PLANNER_MODEL=openai/gpt-4o-mini
SOVEREIGN_AGENT_LLM_EXECUTOR_MODEL=openai/gpt-4o-mini
SOVEREIGN_AGENT_LLM_MEMORY_MODEL=openai/gpt-4o-mini
NEBIUS_SMOKE_MODEL=openai/gpt-4o-mini
```

## Core Runs

```bash
make ex5
make ex5-real
make ex6
make ex6-auto
make ex7
make ex7-real
make ex8-text
make check-submit
```

## Rasa Setup

```bash
make setup-rasa
make ex6-auto
```

Three-terminal mode:

```bash
make rasa-actions
make rasa-serve
make ex6-real
```

After editing `rasa_project/actions/actions.py`, restart `make rasa-actions`.

## Debugging

Most useful commands:

```bash
make verify
make narrate SESSION=<session_id>
make check-submit
uv run pytest tests/public -q
uv run ruff check starter/ grader/ tests/ scripts/
```

Find latest session:

```bash
make logs
```

Inspect a trace:

```bash
cat "<session-dir>/logs/trace.jsonl"
```

## Common Errors

### `402 Payment Required`

The provider key is valid but budget is exhausted.

### `NEBIUS_KEY not set`

The repo is still configured for Nebius, or the key is in the wrong `.env`.

### Rasa training hits Nebius after OpenRouter switch

Check:

```text
rasa_project/endpoints.yml
```

Rasa has its own model groups. It does not automatically use the agent `.env`
model variables.

### `rasa_project/ not found`

The Rasa lifecycle is looking in the wrong directory. It should point to the
repo-local `rasa_project/`.

### Ex5 real does not write flyer

Check the trace:

- did the planner assign work to `structured`?
- did the executor call `generate_flyer`?
- did it call `complete_task` too early?

## Current Validation Snapshot

At the time this folder was created:

```text
uv run pytest tests/public -q
27 passed

make check-submit
Mechanical: 27/27
Behavioural: 19/19
Local non-reasoning: 46/46
```
