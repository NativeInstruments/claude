# kontakt-developer

Claude Code plugin for developing Kontakt instruments and tools.

## Install

```
/plugin install kontakt-developer@native-instruments
```

See the [marketplace README](../../README.md) for adding the marketplace first.

## Commands

### `/setup-mcp [port]`

Registers Kontakt's MCP server with Claude Code so the `komplete-script` dev loop
can talk to a running Kontakt instance.

- **`port`** (optional) — port the Kontakt MCP server listens on. Defaults to `3006`
  if omitted or not a valid integer.

If a server named `kontakt` already exists, you'll be asked whether to overwrite it.

## Skills

Skills load automatically when relevant; no command needed.

### `komplete-script`

Develop Kontakt instrument UIs and tools with **kscript** (Komplete UI) — a
proprietary declarative UI language that is **not in the model's training data**.
The skill bundles the full language reference and loads it on demand before any
kscript is written.

Triggers when you build, edit, or debug a Kontakt instrument UI, work with
`.kscript` files, or mention Komplete UI / kscript / Kontakt Controls / KSP.
