---
name: komplete-script
description: Write Komplete Script (kscript) — the language itself. Covers syntax, types, classes, components, templates, modifiers, modules, reactivity, layout, gestures, the standard library and the ui/math/path/uri/audio_components packages. Use when working with .kscript files or when the user mentions kscript, Komplete Script or Komplete UI. kscript is a proprietary language NOT in AI training data — always load the bundled references before writing any kscript code. Host-independent; for Kontakt instrument UIs (KSP binding, the kontakt package, Kontakt Controls, the Kontakt MCP dev loop) also load the kontakt-developer komplete-script skill.
---

# Komplete Script (kscript)

kscript is a statically typed language with a declarative, reactive UI layer (Komplete UI).
It is **not in your training data** — never write kscript from intuition or by analogy to
Swift/SwiftUI/JavaScript. Ground every construct in the bundled references.

This skill covers the language and its bundled packages only. Anything host-specific — how
a host loads a program, where files must live on disk, how the UI binds to an audio engine,
how compile output is read back — lives in that host's own skill.

For Kontakt instruments that means `kontakt-developer:komplete-script` — KSP binding, the
`kontakt` package, Kontakt Controls, the resource-container layout and the Kontakt MCP dev
loop. Load it alongside this skill for any Kontakt work.

## Reference files — load before writing code

Read these on demand from this skill's `references/` directory:

| File | When to read |
|---|---|
| `references/language.md` | **Always**, before writing any kscript. Syntax, types, variables, functions, classes, components, templates, modifiers, modules, declarative UI, reactivity, layout, gestures. |
| `references/stdlib.md` | Using built-in types: Int, Float, String, Array, Map, Color, Range, Angle, print, Error/Warning. |
| `references/ui-package.md` | Building UI: Components (Text, Rectangle, ZStack, …), Modifiers, enums, constants from the `ui` package. |
| `references/other-packages.md` | `math`, `uri`, `path` and `audio_components` packages. |
| `references/versions.md` | Checking whether a feature exists in the target language version. |
| `references/troubleshooting.md` | Any error/warning, unexpected behavior (flickering, state resets, cycle errors), known issues. |

For a typical UI task the set is `language.md` + `ui-package.md` (+ `stdlib.md` for
collection or string work).

## Language-level project conventions

- One file = one **module**. Filenames must be lowercase (letters, digits, underscores).
- Import paths are dot-separated and **always resolved from the project root** — the host's
  script directory — never relative to the importing file.
- The main module must export the entry point: `export var main: Component = <Component>`.
- Exported types must not refer to un-exported types; annotate the export where needed.

## Version discipline

Features are gated by **language version** (`references/versions.md`). A host maps its own
release to a language version — for Kontakt, `kontakt-developer`'s
`references/kontakt-versions.md` holds that mapping and `kontaktTargetVersion` sets the
ceiling. Establish the target language version before writing code, and never use a feature
newer than it.

## Hard rules

1. **Never invent API.** Every component, modifier, function, parameter label and enum case
   you write must appear in a reference file. If it is not there, say so rather than guessing.
2. The main module must `export var main = <Component>` (type-annotated when required).
3. Respect the target language version — check `references/versions.md` when unsure.
4. Verify code through the host's feedback loop (compile errors, warnings, `print` output)
   before declaring success. Use `print("...")` for temporary debug logging and remove it
   before finishing.
