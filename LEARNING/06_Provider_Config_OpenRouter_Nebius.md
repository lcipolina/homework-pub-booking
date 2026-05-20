# 06 — Provider Configuration: Nebius, OpenRouter, and OpenAI-Compatible APIs

## Overview

The homework defaults to Nebius Token Factory, but the code mostly uses
OpenAI-compatible APIs. That means other providers can work if they expose
compatible chat and embedding endpoints.

In our local setup, Nebius returned:

```text
402 Payment Required: budget exhausted
```

So we switched local real-mode development to OpenRouter.

## Key Environment Variables

The important `.env` variables are:

```env
SOVEREIGN_AGENT_LLM_BASE_URL=https://openrouter.ai/api/v1
SOVEREIGN_AGENT_LLM_API_KEY_ENV=OPENROUTER_API_KEY
SOVEREIGN_AGENT_LLM_PLANNER_MODEL=openai/gpt-4o-mini
SOVEREIGN_AGENT_LLM_EXECUTOR_MODEL=openai/gpt-4o-mini
SOVEREIGN_AGENT_LLM_MEMORY_MODEL=openai/gpt-4o-mini
NEBIUS_SMOKE_MODEL=openai/gpt-4o-mini
```

The variable name `NEBIUS_SMOKE_MODEL` is historical. The smoke script uses it
as the test model even when the base URL is OpenRouter.

## Agent Config vs Rasa Config

There are two config surfaces.

### Agent Config

Used by:

- Ex5 real mode,
- Ex7 loop side if real LLM is used,
- Ex8 manager persona.

Source:

```text
.env
```

### Rasa Config

Used by:

- Ex6 real mode,
- Ex7 real structured half,
- Rasa embeddings during training.

Source:

```text
rasa_project/endpoints.yml
```

Important: changing `.env` is not enough if `endpoints.yml` still hardcodes a
provider. Rasa will follow `endpoints.yml`.

## Why Embeddings Matter

Rasa CALM trains a flow retrieval store. That requires embeddings. If the
embedding model group points to an exhausted provider, `rasa train` fails even
before any conversation happens.

## Real-Mode Failure Types

Provider failures are not always code bugs.

Examples:

- `401 Unauthorized` means key invalid or wrong provider.
- `402 Payment Required` means budget exhausted.
- `404 Not Found` often means wrong model name or base URL.
- timeout means network/provider latency.

## Exam Checklist

You should be able to answer:

1. What does “OpenAI-compatible” mean?
2. Why did `make verify` still work after switching away from Nebius?
3. Why does Rasa need its own provider config?
4. What does a 402 error mean?
5. Why do embeddings matter for Rasa training?
