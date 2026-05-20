# Appendix — Commands, Diagnostics, And Reading The Repo

## The Command Ladder

Use commands in this order when working on the homework:

```text
make
  Shows the walkthrough.

make next
  Tells you the next likely step.

make verify
  Checks environment and provider setup.

make test
  Runs public tests.

make ci
  Runs lint, format check, and tests.

make check-submit
  Runs the local advisory grader.
```

## Exercise Commands

```bash
make ex5
make ex5-real

make ex6
make ex6-auto

make ex7
make ex7-real

make ex8-text
make ex8-voice
```

## Rasa Commands

Manual Rasa mode uses three terminals:

```bash
make rasa-actions
make rasa-serve
make ex6-real
```

Why:

```text
agent process
  │
  ▼
Rasa server
  │
  ▼
Rasa action server
```

If you edit files under:

```text
rasa_project/actions/
```

restart `make rasa-actions`.

## Voice Commands

Text mode:

```bash
make ex8-text
```

Real voice mode:

```bash
make ex8-voice
```

Required keys:

```text
SPEECHMATICS_KEY or SPEECHMATICS_API_KEY
RIME_API_KEY
OPENROUTER_API_KEY
```

When you see:

```text
[turn 1] listening...
```

speak clearly, then pause for about two seconds.

## Reading A Session

Session directories usually contain:

```text
session.json
logs/
  trace.jsonl
  tickets/
workspace/
ipc/
```

Look first at:

```text
logs/trace.jsonl
```

Then inspect tickets:

```text
logs/tickets/<ticket_id>/summary.md
logs/tickets/<ticket_id>/raw_output.json
```

## Useful Search Commands

Find TODOs:

```bash
rg "TODO|NotImplemented|FILL_ME_IN"
```

Find trace events:

```bash
rg "session.state_changed|voice.utterance|executor.tool_called" logs/trace.jsonl
```

Find provider config:

```bash
rg "OPENROUTER|NEBIUS|SPEECHMATICS|RIME" .
```

Find all files in learning docs:

```bash
find LEARNING -maxdepth 1 -type f | sort
```

## Secret Safety

Before pushing:

```bash
git check-ignore -v .env
git status -sb
git diff --cached --name-only
```

Do not commit:

```text
.env
API keys
session artifacts with sensitive data
```

Safe to commit:

```text
source code
answer files
learning notes
assignment checklist
```

## Common Error Meanings

### `SPEECHMATICS_KEY not set`

Voice mode cannot do real STT. Add:

```text
SPEECHMATICS_KEY=...
```

or:

```text
SPEECHMATICS_API_KEY=...
```

### Nebius `402 Payment Required`

The key exists but budget is unavailable/exhausted. Use OpenRouter config if
allowed.

### Rasa Caches Old Action Code

Restart:

```bash
make rasa-actions
```

### Voice Ends Immediately With Silence

You did not speak during the listening window, or the microphone was blocked.

Check macOS microphone permission.

### Local Grader Shows `0/30 Reasoning`

This is normal. Reasoning is CI-only.

Interpret:

```text
46/76 local = 46/46 locally checkable points
```

## Submission Flow

```text
1. Work locally.
2. Run make ci.
3. Run make check-submit.
4. Commit.
5. Push to your fork.
6. Submit fork/commit through course mechanism.
```

Our pushed commit:

```text
e5fb946 Complete pub booking homework
```

If learning docs are expanded later, commit and push again.

