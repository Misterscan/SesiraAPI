# CLI

Run the CLI directly with Sesi:

```bash
sesi --cli -l SesiraAPI/cli.sesi <command>
```

- `init [--force]` creates `sesira.json` and `app.sesi`. Existing files are preserved unless forced.
- `doctor` validates configuration and reports whether the selected provider credential is present.
- `providers` lists adapters, default models, and credential variables.
- `ask <message>` performs a one-shot completion using `sesira.json`.
- `chat` starts a persistent terminal chat. `/memory <query>` shows retrieved context; `/exit` closes it.
- `help` prints command help.

The CLI and library use the same configuration loader, so behavior does not drift between development and deployment.
