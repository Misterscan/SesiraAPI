# Observability

Each application creates a trace ID and emits framework events. Spans cover chat, structured generation, agent runs, tools, workflows, and workflow steps. Model usage and timings are attached when available.

When `log_path` is set, events are appended as one JSON object per line. In-memory events remain available through `app["observer"]["events"]`, and `Sesira.metrics(app["observer"])` aggregates counts, errors, and recorded durations.

Telemetry is synchronous and intentionally small. A production application can rotate the JSONL file externally or pass the event attributes to a custom sink. Do not put prompts, retrieved content, or tool results in event attributes unless the application’s data policy allows it.
