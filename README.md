# Claude Plugins

Claude plugins provided by Native Instruments.

## Plugins

- **komplete-script** — The Komplete Script (kscript) language: syntax, standard library,
  `ui` package. Host-independent.
- **kontakt-developer** — Kontakt instruments and tools: KSP binding, the `kontakt` package,
  Kontakt Controls, the Kontakt MCP dev loop. **Builds on `komplete-script`** — install both
  for Kontakt UI work.

## Install the marketplace

From a local checkout:

```
/plugin marketplace add <path-to-marketplace>
```

Or from remote by URL instead:

```
/plugin marketplace add https://github.com/NativeInstruments/claude.git
```

## Install a plugin

```
/plugin install komplete-script@native-instruments
/plugin install kontakt-developer@native-instruments
```

## Update

```
/plugin marketplace update native-instruments
```
