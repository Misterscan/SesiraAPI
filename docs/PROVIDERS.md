# Providers and streaming

Provider adapters normalize framework settings into Sesi’s native `model()` configuration. They do not duplicate HTTP clients or SDK code.

| Adapter | Default | Credential |
| --- | --- | --- |
| `gemini` | `gemini-3.6-flash` | `GEMINI_API_KEY` |
| `openai` / `gpt` | `gpt-5.6-luna` | `OPENAI_API_KEY` |
| `local` | `local` | none; follows Sesi local-model settings |
| `sesi` | `agent` alias | depends on alias |
| `custom` | required `model` option | provider/runtime-specific |

Every adapter carries `model`, `system`, `temperature`, `max_tokens`, `thinking`, `search`, `cache`, `stream`, and `tools`. Per-request options override adapter values.

Streaming uses Sesi’s native callback bridge. The callback may update a terminal UI, websocket, or accumulator; provider errors still propagate as typed errors. The completed call returns the same response envelope as a non-streaming call.

Provider support ultimately follows the installed Sesi runtime. A custom adapter is appropriate for a model alias or compatible model identifier already understood by `model()`.
