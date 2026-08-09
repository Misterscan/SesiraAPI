# SesiraAPI

SesiraAPI is a production-oriented AI application framework written entirely in Sesi. It builds on Sesi’s native `model()`, streaming, tool, database, and runtime primitives without introducing a heavyweight framework or a JavaScript/Python application layer.

It provides one small, pipe-first API for chatbots, autonomous agents, AI tools, durable memory, retrieval, structured responses, workflows, and operational telemetry.

## Quick start

```bash
# Install the package via sesi install
sesi install Mistercan/SesiraAPI
```
Now import it into your scripts:

```sesi
allow "SesiraAPI" in with Sesira

let config = {
  "provider": "gemini",
  "model": "gemini-3.6-flash",
  "system": "You are a concise product assistant."
}

let app = config | Sesira.create

let answer = "Draft a launch checklist." | Sesira.ask(app)
print answer
```

Run the native test suite:

```bash
npm test
```

Open the CLI:

```bash
# Run chat
npm start 

# Inititate sesira.json
npm run init

# Show help menu
npm run help

# Diagnostics
npm run doctor
```

## What is included

| Capability | Native implementation |
| --- | --- |
| Provider adapters | Gemini, OpenAI/GPT, local inference, Sesi aliases, and custom model names over `model()` |
| Messages and prompts | System/user/assistant/tool messages, multimodal parts, templates, rendering, token estimates |
| Streaming | Function callbacks or stdout streaming with the final accumulated response |
| Structured output | Runtime JSON Schema subset, parsing, validation, and one repair pass |
| Tools and agents | Typed tool descriptors, argument validation, execution results, bounded ReAct loops |
| Short-term memory | Bounded in-process conversation history |
| Long-term memory | Persistent and optionally encrypted `std/db` collections |
| Retrieval | Chunking, metadata filters, lexical relevance ranking, context assembly, durable indexing |
| Workflows | Dependency graphs, conditions, retries, skip/continue policies, function and model steps |
| Observability | Trace IDs, spans, JSONL events, errors, timings, usage attributes, aggregate metrics |
| Configuration | Defaults, `sesira.json`, environment variables, explicit overrides, redaction, validation |
| CLI | Project initialization, diagnostics, providers, one-shot prompts, persistent chat |

## Pipe-first API

The value being operated on is the first parameter, so normal Sesi pipes remain natural:

```sesi
let response = "Hello" | Sesira.chat(app)
let data = "Extract the fields" | Sesira.structured(app, schema)
let memories = "deployment preference" | Sesira.recall(store, {"limit": 3})
let result = task | Sesira.run_agent(agent)
let state = input | Sesira.run_workflow(steps)
```

`chat` returns a response object containing `text`, `provider`, `model`, `usage`, `duration_ms`, and retrieved records. Use `ask` when only the response text is needed.

## Configuration

Resolution order is: built-in defaults → `sesira.json` → environment → explicit values passed to `create`.

```json
{
  "name": "support-bot",
  "provider": "gemini",
  "model": "gemini-3.6-flash",
  "system": "Answer from approved support knowledge.",
  "temperature": 0.2,
  "memory_path": ".sesira/support.db",
  "memory_namespace": "support",
  "observability": true
}
```

Recognized environment overrides include `SESIRA_PROVIDER`, `SESIRA_MODEL`, `SESIRA_MEMORY_PATH`, `SESIRA_MEMORY_NAMESPACE`, `SESIRA_LOG_PATH`, `SESIRA_TEMPERATURE`, and `SESIRA_MAX_TOKENS`. Provider credentials continue to use Sesi’s normal `GEMINI_API_KEY` and `OPENAI_API_KEY` variables.

## Streaming

```sesi
fn on_chunk(chunk) { print chunk }

let result = "Explain the design." | Sesira.stream(app, on_chunk)
print result["usage"]
```

## Structured output

```sesi
let schema = {
  "type": "object",
  "properties": {
    "title": {"type": "string"},
    "priority": {"type": "integer", "minimum": 1, "maximum": 5}
  },
  "required": ["title", "priority"],
  "additionalProperties": false
}

let ticket = "Extract a ticket from: Login is broken and urgent" | Sesira.structured(app, schema)
```

The validator supports objects, arrays, strings, numbers, integers, booleans, nulls, `required`, `enum`, `items`, numeric bounds, string lengths, patterns, and `additionalProperties`.

## Tools and autonomous agents

```sesi
fn lookup(args, context) {
  return "Inventory for " + args["sku"] + ": 14"
}

let inventory = Sesira.tool("inventory", "Look up inventory", {
  "type": "object",
  "properties": {"sku": {"type": "string"}},
  "required": ["sku"],
  "additionalProperties": false
}, lookup)

let agent = Sesira.create_agent(app, {"tools": [inventory], "max_steps": 6})
let result = "Check inventory for SKU-42 and summarize." | Sesira.run_agent(agent)
```

Agent loops are bounded. Every tool call is schema-validated and returns an explicit `ok`, `result`/`error`, and duration rather than hiding failures.

## Memory and retrieval

```sesi
let store = Sesira.long_memory("knowledge.db", "docs")
"Sesi is pipe-friendly." | Sesira.remember(store, {"source": "guide"})
let hits = "pipe syntax" | Sesira.recall(store, {"limit": 3})
```

Pass a password as the third argument to `long_memory` for Sesi’s AES-256-CBC database encryption. Retrieval is local and deterministic; it uses token coverage plus a phrase bonus and never requires an embedding provider.

For RAG, index documents into the application memory and chat normally:

```sesi
let docs = [Sesira.document(read_file("handbook.md"), {"source": "handbook"}, "handbook")]
docs | Sesira.index_documents(app, {"size": 1200, "overlap": 150})
print "What is the refund window?" | Sesira.ask(app)
```

## Workflows

Workflows are dependency graphs. A step receives its dependency output (or an object of dependency outputs) plus the complete workflow state.

```sesi
fn verify(value, state) { return {"approved": len(value) > 20, "draft": value} }

let steps = [
  Sesira.prompt_step("draft", app["provider"], "Draft a brief for {{input}}."),
  Sesira.step("verify", verify, {"depends_on": ["draft"]})
]

let result = "New Sesi package" | Sesira.run_workflow(steps, {"observer": app["observer"]})
```

## Package layout

```text
SesiraAPI/
├── index.sesi                 public API
├── cli.sesi                   native CLI
├── src/
│   ├── core.sesi              application runtime
│   ├── providers.sesi         inference adapters
│   ├── messages.sesi          messages and templates
│   ├── schema.sesi            structured-output validation
│   ├── tools.sesi             tool registry/execution
│   ├── agents.sesi            autonomous agent loop
│   ├── memory.sesi            short/long-term memory
│   ├── retrieval.sesi         chunking and retrieval
│   ├── workflow.sesi          orchestration
│   ├── observability.sesi     tracing and metrics
│   └── config.sesi            configuration
├── examples/
├── tests/
└── docs/
```

See [API.md](docs/API.md), [ARCHITECTURE.md](docs/ARCHITECTURE.md), and the focused guides in `docs/` for operational details.

## Production guidance

- Put `.sesira/` on persistent storage and back up the database.
- Set a database password when memory contains sensitive data.
- Keep agent `max_steps` bounded and expose only least-privilege tool handlers.
- Use `additionalProperties: false` for tool and structured-output schemas.
- Keep JSONL telemetry outside public web roots and apply normal log retention.
- Review retrieved sources before enabling high-impact actions; retrieval relevance is not authorization.

SesiraAPI is licensed under MIT.
