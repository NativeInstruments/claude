---
name: komplete-script
description: Build Kontakt instrument UIs with Komplete UI — the Kontakt layer on top of kscript. Covers KSP control binding, the kontakt package, the Kontakt Controls package, resource-container layout, kontaktTargetVersion, and the Kontakt MCP dev loop. Use when the user wants to build, edit or debug a Kontakt instrument UI, works on an .nki or its komplete_scripts folder, or asks to connect a UI to KSP. Requires the kscript-developer komplete-script skill for the language itself — load it first.
---

# Komplete UI for Kontakt

This skill covers the **Kontakt-specific** layer: how a kscript UI lives inside an
instrument, how it binds to KSP, and how to verify it through Kontakt's MCP server.

## Step 1 — load the language skill first

The kscript language (syntax, types, components, modifiers, `ui` package, stdlib) is **not
in this skill**. Before writing any kscript, invoke the `kscript-developer:komplete-script`
skill and read its references.

If that skill is unavailable, tell the user to install it:

```
/plugin install kscript-developer@native-instruments
```

Do **not** write kscript from memory — the language is not in your training data.

## Reference files

Read these on demand from this skill's `references/` directory:

| File | When to read |
|---|---|
| `references/kontakt-package.md` | Connecting to Kontakt/KSP: KSP control classes (KSPSlider, KSPKnob, …), Help modifier, zones, groups, samples, programs. Built into Kontakt — always available. |
| `references/kontakt-integration.md` | The binding pattern: how a component reads and writes a KSP control's value. |
| `references/kontakt-controls.md` | Using the optional Kontakt Controls package (ready-made Slider, Knob, XYPad, Stepper, Switch, ToggleButton). See install note below. |
| `references/kontakt-versions.md` | Kontakt → kscript version mapping, `kontaktTargetVersion`, Kontakt-only feature availability. |
| `references/ksp-migration.md` | Porting an existing KSP UI, or explaining Komplete UI to a KSP developer. |

For a task like "add a knob controlling filter cutoff": `kscript-developer:komplete-script` → `language.md` +
`ui-package.md`, plus `kontakt-package.md` and `kontakt-integration.md` here (+
`kontakt-controls.md` if the package is installed).

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

- The main module is selected in Kontakt's *Instrument Options* dialog (KOMPLETE UI section),
  which also sets instrument width/height.
- Import paths resolve from `komplete_scripts/`, never relative to the importing file.

## Version ceiling

`kontaktTargetVersion` in `Resources/info/library.json` is the ceiling. Read it, map it to a
kscript language version via `references/kontakt-versions.md`, and treat that language
version as the limit when using anything from the language references. If a feature seems
missing, check both tables before assuming an error.

## Kontakt Controls package (optional)

Beginner-friendly controls that bind to KSP controls via `control_id`. **Not built in** — the
user must download it manually and copy its folder into `Resources/komplete_scripts/`. Before
using it, verify the `kontakt_controls/` folder exists next to `main.kscript` (existence check
only — do not read the folder's source to learn the API). If missing, tell the user to
download it from the Native Instruments homepage and copy it in. Import:
`import * from kontakt_controls`.

**API source of truth is `references/kontakt-controls.md`, not the installed folder.** Read
the md for every control's API. Only fall back to reading the actual `kontakt_controls/`
source when Kontakt reports an API error (unknown symbol, wrong parameter, missing modifier)
on code that matches the md — that signals the installed package version has drifted from the
reference. When that happens, note the mismatch to the user.

## KSP connection basics

Instrument logic lives in KSP (Kontakt's script editor); Komplete UI is the view layer. KSP
declares UI controls and calls `expose_controls` in `on init`; kscript binds to them by
control id (KSP name without the `$`):

```
on init
    declare ui_slider $my_slider (0, 100)
    expose_controls
end on
```

KSP connections are fixed at load time, so **declare them globally**, never inside a
component. Details and per-control classes: `references/kontakt-package.md`; the full binding
pattern: `references/kontakt-integration.md`.

## Development loop (Kontakt MCP server)

Kontakt exposes an MCP server that closes the feedback loop: **saving a .kscript file makes
Kontakt reload the instrument automatically** — no manual reload step. Log output (compile
errors, warnings, `print(...)` output) is fetched via MCP. Depending on whether you're editing
an instrument or a tool, use `instrument_get_kscript_messages` or `tool_get_kscript_messages`.

Tools (load schemas via ToolSearch first if deferred):
- `list_instruments` — instruments loaded in the rack. Call once at session start to get the
  instrument id. Usually exactly one instrument during development; if several, ask the user
  which one.
- `list_tools` — tools loaded in the rack
- `instrument_get_kscript_messages` — errors, warnings and `print` output for an instrument.
  Call after every edit.
- `tool_get_kscript_messages` — errors, warnings and `print` output for a tool. Call after
  every edit.

If the server is not registered, run `/setup-mcp` (see the plugin README).

### The loop — run until clean

After **every** edit to a `.kscript` file:

1. Save the file (the Write/Edit tool does this). Kontakt reloads automatically.
2. Call `instrument_get_kscript_messages` for an instrument, or
   `tool_get_kscript_messages` for a tool.
3. Errors present → match against `kscript-developer:komplete-script` → `references/troubleshooting.md`
   (Common Errors), fix, repeat from 1. Do not ask the user between iterations.
4. Warnings → fix if clearly yours; report otherwise.
5. Clean → done. Report the result to the user.

If messages look stale or empty when an error is expected, re-fetch once; if the MCP server is
unreachable, tell the user to check that Kontakt is running with Developer Mode enabled
(Options → Developer tab).

Use `print("...")` in kscript for temporary debug logging — output arrives through the same
MCP call. Remove debug prints before finishing.

## Working with non-programmer users

The target user often cannot code. Therefore:

- Translate their intent ("I want a big filter knob on the left") into kscript yourself; never
  ask them to write or read code.
- Explain results in UI terms ("the knob now sits in the top-left and controls the filter"),
  not code terms.
- When their request is ambiguous (placement, size, color, behavior), ask a short concrete
  question with options rather than guessing.
- Anything that must happen inside Kontakt's own UI (creating the resource container, setting
  the main module, instrument size, editing KSP in the Script Editor, enabling Developer Mode)
  you cannot do for them — give exact click-path instructions.
- Verify every change through the MCP loop before telling them it works.

## Hard rules

1. **Load `kscript-developer:komplete-script` before writing kscript.** Never invent API —
   every component, modifier, function, parameter label and enum case must appear in a
   reference file.
2. All `.kscript` files must live under `Resources/komplete_scripts/`.
3. The main module must `export var main = <Component>`.
4. KSP connections are declared globally, not inside components.
5. Run the MCP feedback loop after every edit; never declare success on unverified code.
6. Respect `kontaktTargetVersion` — don't use features newer than the mapped language version.
