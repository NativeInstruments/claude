# kscript-developer

Claude Code plugin for developing with **Komplete Script** (kscript) — the language behind
Komplete UI.

## Install

```
/plugin install kscript-developer@native-instruments
```

See the [marketplace README](../../README.md) for adding the marketplace first.

## Skills

Skills load automatically when relevant; no command needed.

### `komplete-script`

The kscript language itself: syntax, types, classes, components, templates, modifiers,
modules, reactivity, layout, gestures, the standard library, and the `ui`, `math`, `path`,
`uri` and `audio_components` packages. kscript is **not in the model's training data** — the
skill bundles the full reference and loads it on demand before any kscript is written.

Triggers when you work with `.kscript` files or mention kscript, Komplete Script or
Komplete UI.

The skill is **host-independent**. Building a Kontakt instrument UI additionally needs
[`kontakt-developer`](../kontakt-developer/README.md), which adds KSP binding, the `kontakt`
package, Kontakt Controls, the Kontakt MCP dev loop, and the Kontakt → kscript version
mapping.
