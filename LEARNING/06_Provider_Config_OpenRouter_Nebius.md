# 06 — Provider Config: OpenRouter, Nebius, Keys, And Models

## What Is A Model Provider?

An LLM provider is a service that hosts models behind an API.

Examples:

```text
OpenAI
Nebius
OpenRouter
Anthropic
Google
```

Your code does not run the model locally. It sends a request to a provider:

```text
your Python code ──HTTPS──▶ provider API ──▶ model ──▶ response
```

## What Is An API Key?

An API key is a secret token that proves you are allowed to use a provider.

```text
API key = password-like credential for a service
```

It should live in `.env`, never in source code.

Good:

```text
.env
  OPENROUTER_API_KEY=...
```

Bad:

```python
api_key = "real-secret-here"
```

The grader can penalize committed secrets.

## What Is `.env`?

`.env` is a local configuration file.

It contains machine-specific secrets and settings:

```text
OPENROUTER_API_KEY=...
RASA_PRO_LICENSE=...
SPEECHMATICS_API_KEY=...
RIME_API_KEY=...
```

The repo ignores `.env` via `.gitignore`, so it should not be pushed.

## Environment Variables

Programs read config from environment variables:

```python
os.environ["OPENROUTER_API_KEY"]
```

`.env` is a convenient way to load those values during local development.

Infographic:

```text
.env file
  │
  ▼
python-dotenv / config loader
  │
  ▼
os.environ
  │
  ▼
LLM client / Rasa / Speechmatics / Rime
```

## What Is A Base URL?

Many model providers expose OpenAI-compatible APIs. That means the request
shape looks like OpenAI's API, but the URL points somewhere else.

OpenAI-like request:

```text
POST /chat/completions
```

Provider-specific base:

```text
https://api.openai.com/v1
https://api.studio.nebius.com/v1
https://openrouter.ai/api/v1
```

Base URL + endpoint:

```text
https://openrouter.ai/api/v1/chat/completions
```

## Why OpenRouter Worked Here

The homework originally expected Nebius.

But the available Nebius key returned budget/payment errors. Instead of
rewriting the whole system, we configured an OpenAI-compatible provider:

```text
SOVEREIGN_AGENT_LLM_BASE_URL=https://openrouter.ai/api/v1
SOVEREIGN_AGENT_LLM_API_KEY_ENV=OPENROUTER_API_KEY
SOVEREIGN_AGENT_LLM_PLANNER_MODEL=openai/gpt-4o-mini
SOVEREIGN_AGENT_LLM_EXECUTOR_MODEL=openai/gpt-4o-mini
SOVEREIGN_AGENT_LLM_MEMORY_MODEL=openai/gpt-4o-mini
```

The code asks the config:

```text
"Which base URL? Which key env var? Which model?"
```

rather than hardcoding Nebius everywhere.

## Provider Abstraction

Provider abstraction means your application code depends on a common interface,
not one provider's details.

Bad:

```python
client = NebiusClient(api_key="...")
```

Better:

```python
config = Config.from_env()
client = OpenAICompatibleClient(
    base_url=config.base_url,
    api_key=config.api_key,
)
```

Then switching providers is configuration, not a rewrite.

## Model Names

Model names are provider-specific identifiers:

```text
openai/gpt-4o-mini
meta-llama/...
qwen/...
```

The same model family may have different names across providers.

That is why config needs:

```text
planner model
executor model
memory model
```

Different subcomponents may use different models in larger systems.

## Why Real Mode Can Fail Even If Code Is Correct

Real mode depends on external services.

Failures can come from:

- expired key,
- empty key,
- exhausted budget,
- provider outage,
- model unavailable,
- rate limit,
- network failure,
- malformed base URL.

That is different from a Python bug.

```text
Code bug:
  TypeError, wrong path, bad validation logic.

Provider problem:
  401 unauthorized, 402 payment required, 429 rate limit.
```

The homework intentionally exposes you to this distinction.

## Rasa Provider Configuration

Rasa can need:

- an LLM model group,
- an embeddings model group.

The repo's `rasa_project/endpoints.yml` was adjusted to use OpenRouter with
the existing group names. This avoided changing Rasa flow code while swapping
the provider underneath.

Mental model:

```text
Rasa config name:
  nebius_llm

Actual provider:
  OpenRouter

Why okay?
  The symbolic group name can stay stable while the implementation changes.
```

## Voice Provider Configuration

Ex8 uses separate providers:

```text
Speechmatics -> speech-to-text
Rime         -> text-to-speech
OpenRouter   -> manager persona LLM
```

These are different services with different keys.

```text
Speechmatics key:
  allows transcription

Rime key:
  allows audio generation

OpenRouter key:
  allows LLM replies
```

## Secret Safety Checklist

Before pushing:

```text
git check-ignore -v .env
git diff --cached --name-only
rg "API_KEY=|RIME_API_KEY=|OPENROUTER_API_KEY=" .
```

You want:

- `.env` ignored,
- no real keys in source files,
- only placeholder examples in docs.

## Repo Files To Study

- `.env`
  - local secrets, not committed.

- `rasa_project/endpoints.yml`
  - Rasa provider config.

- `starter/voice_pipeline/manager_persona.py`
  - LLM provider config for Ex8 persona.

- `starter/edinburgh_research/run.py`
  - real-mode LLM behavior.

- `scripts/educator_diagnostics.py`
  - environment diagnostics.

## The Big Lesson

Provider configuration is infrastructure, not business logic.

Good agent code should be able to say:

```text
"Use whatever OpenAI-compatible provider the environment config selects."
```

That is what let the homework proceed when Nebius was unavailable.

