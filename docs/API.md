# API reference

All examples assume `allow "SesiraAPI" in with Sesira`.

## Application

- `create(config = {}) -> object` creates an application with normalized config, provider, optional memory bank, and observer.
- `complete(prompt, adapter, options = {}) -> object` calls a provider without creating an application.
- `chat(input, app, options = {}) -> object` returns the complete response envelope.
- `ask(input, app, options = {}) -> string` returns response text.
- `stream(input, app, on_chunk, options = {}) -> object` streams chunks and returns the completed envelope.
- `structured(prompt, app, schema, options = {}) -> any` generates, parses, validates, and, if needed, repairs JSON.

The pipe value is always the first argument: `"hello" | Sesira.ask(app)`.

## Providers

- `provider(kind, options = {}) -> object`
- `supported_providers() -> array`

Kinds are `gemini`, `openai`/`gpt`, `local`, `sesi`, and `custom`.

## Messages

- `message(role, content, metadata = {})`
- `system(content)`, `user(content)`, `assistant(content)`
- `tool_result(name, content, call_id = "")`
- `text_part(text)`, `image_part(path)`, `audio_part(path)`
- `render_prompt(template, values)`

## Tools and agents

- `tool(name, description, parameters, handler) -> object`
- `execute_tool(call, tools, context = {}) -> object`
- `tool_schemas(tools, provider_name = "gemini") -> array`
- `register_tools(tools) -> array`
- `create_agent(app, options = {}) -> object`
- `run_agent(task, agent, options = {}) -> object`

Tool handlers receive `(args, context)`. Agent results contain `ok`, `answer`, `steps`, and `step_count`.

## Memory and retrieval

- `short_memory(options = {})`
- `long_memory(path, namespace = "default", password = null)`
- `memory_bank(options = {})`
- `add_short(item, store)`, `recent_memory(store, limit = 0)`, `clear_short_memory(store)`
- `remember(content, store, metadata = {}, id = "")`
- `recall(query, store, options = {})`
- `get_memory(id, store)`, `list_memory(store, limit = 0)`, `forget(id, store)`, `clear_long_memory(store)`
- `retrieve(query, app, options = {})`
- `document(content, metadata = {}, id = "")`
- `chunk(document, options = {})`
- `search(documents, query, options = {})`
- `context(results, options = {})`
- `index_documents(documents, app, options = {})`

Retrieval options include `limit`, `min_score`, and exact-match `filter`. Chunk options include `size` and `overlap` in characters.

## Workflows

- `step(id, handler, options = {})`
- `prompt_step(id, adapter, prompt, options = {})`
- `run_workflow(input, steps, options = {})`

Step options: `depends_on`, `when`, `retries`, and `on_error`. Prompt steps also accept `model_options`.

## Observability

- `observer(options = {})`
- `emit(observer, name, attributes = {})`
- `start_span(observer, name, attributes = {})`, `end_span(observer, span, attributes = {})`
- `metrics(observer)`

Observer options are `enabled`, `path`, and `trace_id`.

## Configuration and validation

- `load_config(path = "sesira.json", overrides = {})`
- `validate(value, schema)`
