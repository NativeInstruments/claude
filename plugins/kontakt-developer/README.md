# kontakt-developer

Claude Code plugin for developing Kontakt instruments and tools.

## Install

```
/plugin install kontakt-developer@native-instruments
```

See the [marketplace README](../../README.md) for adding the marketplace first.

## Prerequisite

This plugin covers the **Kontakt layer** only. The kscript language itself lives in
[`kscript-developer`](../kscript-developer/README.md) — install it too:

```
/plugin install kscript-developer@native-instruments
```

Without it the `komplete-script` skill here has no language reference to load and will tell
you to install it rather than writing kscript.

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

The Kontakt-specific layer of Komplete UI: KSP control binding, the built-in `kontakt`
package, the optional Kontakt Controls package, resource-container layout,
`kontaktTargetVersion` and its mapping to a kscript language version, and the Kontakt MCP
dev loop.

Triggers when you build, edit or debug a Kontakt instrument UI, work on an `.nki` or its
`komplete_scripts` folder, or mention KSP / Kontakt Controls.

It loads `kscript-developer:komplete-script` first for the language itself.
