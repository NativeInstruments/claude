---
name: komplete-script
description: Develop Kontakt instrument UIs and tools with kscript (Komplete UI). Use when the user wants to build, edit, or debug a Kontakt instrument UI, works with .kscript files, mentions Komplete UI, kscript, Kontakt Controls, or asks to connect a UI to KSP. kscript is a proprietary language NOT in AI training data — always load the bundled references before writing any kscript code.
---

# Komplete Script for Kontakt

kscript (Komplete Script) is a declarative UI language for building Kontakt instrument interfaces. It is **not in your training data** — never write kscript from intuition or by analogy to Swift/SwiftUI/JavaScript. Always ground every construct in the bundled references.

## Reference files — load before writing code

Read these on demand from this skill's `references/` directory:

| File | When to read |
|---|---|
| `references/language.md` | **Always**, before writing any kscript. Syntax, types, variables, functions, classes, components, templates, modifiers, modules, declarative UI, layout, gestures. |
| `references/stdlib.md` | Using built-in types: Int, Float, String, Array, Map, Color, Range, Angle, print, Error/Warning. |
| `references/ui-package.md` | Building UI: Components (Text, Rectangle, ZStack, …), Modifiers, enums, constants from the `ui` package. |
| `references/kontakt-package.md` | Connecting to Kontakt/KSP: KSP control classes (KSPSlider, KSPKnob, …), zones, samples, programs. Built into Kontakt — always available. |
| `references/kontakt-controls.md` | Using the optional Kontakt Controls package (ready-made Slider, Knob, XYPad, Stepper, Switch, ToggleButton). See install note below. |
| `references/other-packages.md` | Math, URI, Filesystem, Audio Components packages. |
| `references/troubleshooting.md` | Any error/warning in logs, unexpected behavior, KSP→Komplete UI migration, feature availability by Kontakt version. |

For a task like "add a knob controlling filter cutoff", the typical set is: `language.md` + `ui-package.md` + `kontakt-package.md` (+ `kontakt-controls.md` if the package is installed).

## Project structure

A Kontakt instrument with Komplete UI looks like:

```
Instruments/
├── your-instrument.nki
Resources/
├── komplete_scripts/        ← ALL .kscript files live here
│   ├── main.kscript         ← main module, must `export var main = <Component>`
│   └── kontakt_controls/    ← optional, manually installed package
└── info/library.json        ← kontaktTargetVersion gates available features
```

- The main module is selected in Kontakt's *Instrument Options* dialog (KOMPLETE UI section), which also sets instrument width/height.
- `kontaktTargetVersion` in `Resources/info/library.json` determines the available Komplete UI feature set. Check `references/troubleshooting.md` for the version table if a feature seems missing.

### Kontakt Controls package (optional)

Beginner-friendly controls that bind to KSP controls via `control_id`. **Not built in** — the user must download it manually and copy its folder into `Resources/komplete_scripts/`. Before using it, verify the `kontakt_controls/` folder exists next to `main.kscript` (existence check only — do not read the folder's source to learn the API). If missing, tell the user to download it from the Native Instruments homepage and copy it in. Import: `import * from kontakt_controls`.

**API source of truth is `references/kontakt-controls.md`, not the installed folder.** Read the md for every control's API. Only fall back to reading the actual `kontakt_controls/` source when Kontakt reports an API error (unknown symbol, wrong parameter, missing modifier) on code that matches the md — that signals the installed package version has drifted from the reference. When that happens, note the mismatch to the user.

### KSP connection basics

Instrument logic lives in KSP (Kontakt's script editor); Komplete UI is the view layer. KSP declares UI controls and calls `expose_controls` in `on init`; kscript binds to them by control id (KSP name without the `$`):

```
on init
    declare ui_slider $my_slider (0, 100)
    expose_controls
end on
```

Details and per-control classes: `references/kontakt-package.md`.

## Development loop (Kontakt MCP server)

Kontakt exposes an MCP server that closes the feedback loop: **saving a .kscript file makes Kontakt reload the instrument automatically** — no manual reload step. Log output (compile errors, warnings, `print(...)` output) is fetched via MCP. Depending on whether you're editing an instrument or a tool, use `instrument_get_kscript_messages` or `tool_get_kscript_messages`.

Tools (load schemas via ToolSearch first if deferred):
- `list_instruments` — instruments loaded in the rack. Call once at session start to get the instrument id. Usually exactly one instrument during development; if several, ask the user which one.
- `list_tools` — tools loaded in the rack
- `instrument_get_kscript_messages` — errors, warnings, and `print` output for an instrument. Call after every edit.
- `tool_get_kscript_messages` — errors, warnings, and `print` output for an tool. Call after every edit.

### The loop — run until clean

After **every** edit to a `.kscript` file:

1. Save the file (the Write/Edit tool does this). Kontakt reloads automatically.
2. Call `get_instrument_kscript_messages` for the instrument or `tool_get_kscript_messages` for the tool.
3. Errors present → match against `references/troubleshooting.md` (Common Errors), fix, repeat from 1. Do not ask the user between iterations.
4. Warnings → fix if clearly yours; report otherwise.
5. Clean → done. Report result to the user.

If messages look stale or empty when an error is expected, re-fetch once; if the MCP server is unreachable, tell the user to check that Kontakt is running with Developer Mode enabled (Options → Developer tab).

Use `print("...")` in kscript for temporary debug logging — output arrives via `get_instrument_kscript_messages`. Remove debug prints before finishing.

## Working with non-programmer users

The target user often cannot code. Therefore:

- Translate their intent ("I want a big filter knob on the left") into kscript yourself; never ask them to write or read code.
- Explain results in UI terms ("the knob now sits in the top-left and controls the filter"), not code terms.
- When their request is ambiguous (placement, size, color, behavior), ask a short concrete question with options rather than guessing.
- Anything that must happen inside Kontakt's own UI (creating the resource container, setting the main module, instrument size, editing KSP in the Script Editor, enabling Developer Mode) you cannot do for them — give exact click-path instructions (see `references/troubleshooting.md` and the setup notes above).
- Verify every change through the MCP loop before telling them it works.

## Hard rules

1. **Never invent API.** Every component, modifier, function, parameter label, and enum case you write must appear in a reference file. If it's not there, say so and check `references/troubleshooting.md` / ask the user, rather than guessing.
2. All `.kscript` files must live under `Resources/komplete_scripts/`.
3. The main module must `export var main = <Component>`.
4. Run the MCP feedback loop after every edit; never declare success on unverified code.
5. Respect `kontaktTargetVersion` — don't use features newer than the target version.
