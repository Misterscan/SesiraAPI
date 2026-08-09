# Architecture

SesiraAPI is a set of ordinary `.sesi` modules. The public `index.sesi` delegates to small focused modules; applications can use the public facade or import a focused module directly.

The runtime path for a chat request is:

```text
message input
  → normalize/render messages
  → retrieve relevant persistent memory
  → build system and conversation context
  → provider adapter → Sesi model()
  → capture model_usage()
  → persist user/assistant turns
  → close trace span and return response
```

There is no framework-owned network client or provider SDK. Sesi’s runtime remains the source of truth for credentials, aliases, local inference, streaming, caching, search grounding, and model usage.

## Module boundaries

- `core.sesi` composes the application and owns chat request flow.
- `providers.sesi` is the only module that directly invokes `model()`.
- `messages.sesi` is provider-neutral.
- `schema.sesi` validates runtime JSON Schema descriptors.
- `tools.sesi` validates arguments before invoking first-class Sesi functions.
- `agents.sesi` implements a bounded ReAct state machine.
- `memory.sesi` stores recent state in memory and durable state in `std/db`.
- `retrieval.sesi` is pure deterministic text retrieval.
- `workflow.sesi` runs dependency graphs and does not depend on agents.
- `observability.sesi` records framework events without changing execution semantics.

## Failure model

Provider, configuration, structured-output, agent, memory, and workflow failures use Sesi typed errors. Tool handlers return explicit error envelopes so an agent can observe and recover from a failed action. Workflows default to fail-fast; individual steps may use `on_error: "continue"`.

## Dependency policy

SesiraAPI has no package dependencies. It uses Sesi core functions plus `std/db` and `std/terminal`. The application stays compatible with the same platforms and provider support as the installed Sesi runtime.
