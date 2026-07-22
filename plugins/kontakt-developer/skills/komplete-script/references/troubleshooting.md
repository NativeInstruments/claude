<!-- Condensed from Komplete UI docs (Common Errors, Known Issues, FAQ, KSP migration, Changelog) -->

# Troubleshooting & Migration Reference

## Common Errors

### Error: `unwrapping a nil value`

Cause: `!` force-unwrap on an optional that was `nil` at runtime (e.g. map lookup for a missing key returns nil).

```kscript
var values = ["a": 0, "b": 1, "c": 2]
var fails = values["missing_key"]! // error — map lookup returns nil
```

Fix: check for `nil` before unwrapping, or use a ternary fallback:

```kscript
var result = values["missing_key"] != nil ? values["missing_key"]! : -1

if values["missing_key"] != nil {
    print("Found: \{values["missing_key"]!}")
}
```

### Components recreated / flickering / state resets (creating components inside a function)

Symptom: components are recreated repeatedly, causing flickering or unexpected state resets. No error message.

Cause: calling a function inside a reactive expression (property, computed state, component body) tracks the function's *arguments* as dependencies. When arguments change, the function reruns — if it creates components, new instances replace old ones.

```kscript
fun make_item(text: String) -> (Component) {
    return Text(text)    // new Text created every time text changes
}
```

Fix: use a `template` instead — templates forward parameters lazily and create no reactive dependency on their arguments:

```kscript
var make_item = template (text: String) {
    Text(text)           // stable — not recreated when text changes
}
```

## Known Issues

- **Differently composed UTF-8 characters may not compare equal.** Pre-composed vs decomposed forms look identical but can compare unequal in string comparison. (Several string methods were fixed in 8.2; comparison itself remains a known issue.)
- **Spacer size incorrect if modifiers are applied to it.** A modifier applied to a `Spacer` in a stack can alter its size unexpectedly. Workaround: avoid applying modifiers directly on `Spacer`.
- **Indexing the iterated container inside a declarative for loop** (fixed in Kontakt 8.4): accessing `presets[i]` inside `for i, preset in presets.enumerated() { ... }` can cause out-of-bounds errors when elements are removed. Workaround (pre-8.4): use only the iteration values (`preset`), not `presets[i]`.
- **Same state in a declarative for loop's sequence and its body → "cycle detected" error** (fixed in Kontakt 8.4): e.g. using `self.offset` both in `presets.subsequence(from: self.offset, length: 1)` and in the loop body. Workaround (pre-8.4): embed the total index/id into the data itself and avoid using the state inside the body:

```kscript
class Preset {
    id: Int
    name: String
}
// iterate: for preset in presets.subsequence(from: self.offset, length: 1) { Text("\{preset.id}: \{preset.name}") }
```

## FAQ

- **Komplete UI vs Komplete Script?** Komplete UI = the UI framework (the `ui` package). Komplete Script = the underlying programming language used in `.kscript` files (usable for general scripting too).
- **Reporting issues?** Join the NI Developer Slack (invite via builder-experience-team@native-instruments.com).

## KSP → Komplete UI Migration

Komplete Script mixes imperative code (logic: variables, functions, loops) and declarative code (UI: components, modifiers, layout). Unlike KSP's global step-by-step style, the UI is described as what it should look like given state (reactivity) — no manual update logic.

### Variables

Declared anywhere in imperative scope, must be initialized (no defaults), type usually inferred:

```txt
declare $test := -1
```
```kscript
var test_implicit = -1
var test_explicit: Int = -1 // optional explicit type
```

### Arrays and Maps

No type label, no fixed size, dynamic growth, multidimensional supported:

```txt
declare %presets[10 * 3] := ( ...
    { 1 }  8, 8, 8, 0,  0, 0, 0, 0, ...
```
```kscript
var presets = [
    [8, 8, 8, 0, 0, 0, 0, 0],
    [8, 8, 8, 8, 0, 0, 0, 0],
]
var empty: [[Int]] = [[]] // explicit type required when initializing empty
empty.append([0, 0, 5, 3, 2, 0, 0, 0])
```

Maps (no KSP equivalent) — key-value lookup returns an optional:

```kscript
var preset_data = ["Warm": ["cutoff": 80, "resonance": 20]]
var warm = preset_data["Warm"] // optional — nil if no match
if warm != nil {
    var cutoff = warm!["cutoff"]
}
```

### Functions

```txt
function do_something()
    { body }
end function
call do_something
```
```kscript
fun do_something() { /* body */ }
do_something()

// arguments + multiple return values:
fun split_name(full_name: String) -> (String, String) {
    var parts = full_name.split(separator: " ")
    return parts[0], parts[1]
}
var first, last = split_name(full_name: "Maria Philipps")
```

`return` exits the function like `exit` in KSP, but can also carry values.

### Entry point

No `on init`. The main module (linked in Instrument Options) runs once at instrument load — its top-level scope is the `on init` equivalent. The main module MUST export a UI component:

```kscript
import { Text } from ui
export var main = Text("Hello World") // required in the main module
```

### Project structure

Any number of modules (files); no script-slot limit. All code goes in the instrument's Resource Container under the `komplete_scripts` folder; import modules from there.

### UI elements

Import the UI package: `import * from ui`. Components are more basic than KSP widgets (Rectangle, Text, …) and are composed. KSP UI controls remain the data bridge to the Kontakt Engine — the `kontakt` package connects Komplete UI components to KSP controls. The optional `kontakt_controls` package provides ready-made KSP-connected components (Slider, Knob, …):

```kscript
import * from ui
import * from kontakt_controls

export component Main {
    ZStack {
        Rectangle(color: Color(0xFF2A2A2A)) // background
        Slider(control_id: "my_slider", label: "My Slider") // control_id = KSP ui control id
    }
}
export var main = Main()
```
```txt
on init
    declare ui_slider $my_slider (0, 100)
    set_control_par(get_ui_id($my_slider), $CONTROL_PAR_DEFAULT_VALUE, 0)
    expose_controls
end on
```

Declarative for loop replaces repetitive declarations:

```kscript
HStack {
    for text in ["Pan", "Vibrato", "Tune"] {
        Slider(control_id: text, label: text)
    }
}
```

### Callbacks and reactivity

KSP callbacks are global (`on ui_control`, `on ui_update`); Komplete UI callbacks are attached to specific components/modifiers (predefined ones like Canvas or DragGesture, or your own function properties):

```txt
on init
    declare ui_text_edit @label_name
    set_control_par_str(get_ui_id(@label_name), $CONTROL_PAR_TEXT, "Edit me")
end on
on ui_control (@label_name)
    message(@label_name & " it is!")
end on
```
```kscript
import { TextInput } from ui

export component Main {
    text: String = "Edit me"
    TextInput(self.$text, on_submitted: fun () {
        print("\{self.text} it is!")
    })
}
export var main = Main()
```

Reactivity: describe the UI as a function of state; no manual show/hide (`$CONTROL_PAR_HIDE`) logic:

```kscript
import * from ui

component Button {
    @binding syncing: Bool
    Text("Sync") with {
        Padding(5)
        Background {
            Rectangle(color: self.syncing ? Color(0x30000000) : Color(0x70000000), radius: 2)
        }
        TapGesture(fun (event) { self.syncing = not self.syncing })
    }
}

export component Main {
    syncing: Bool = false
    VStack {
        Button(syncing: self.$syncing)
        if self.syncing {
            Text("Syncing...") // shown/hidden automatically with state
        }
    }
}
export var main = Main()
```

### Layout

Container components replace pixel placement: `VStack` (vertical), `HStack` (horizontal), `ZStack` (overlapping). Absolute positioning is still possible with the `Position` modifier inside a ZStack (analog of `ui_panel` + `CONTROL_PAR_POS_X/Y`):

```kscript
component Bar {
    ZStack {
        Knob(control_id: "attack", label: "Attack") with { Position(x: 0, y: 0) }
        Knob(control_id: "decay", label: "Decay") with { Position(x: 50, y: 0) }
    }
    with {
        Frame(width: 200, height: 70) // ZStack size
        Position(x: 100, y: 20)       // ZStack position in instrument UI
    }
}
```

Note: instrument UI size is set in Instrument Options → Instrument; no `make_perfview` needed.

## Feature availability by Kontakt version

| Version | Notable additions / changes |
|---|---|
| 8.0 (beta) | Komplete UI public beta; `komplete_scripts` folder in resource container; loaded via `load_komplete_ui("module_name")` from KSP |
| 8.1 (beta) | `KSPTextEdit`, `KSPLabel` |
| 8.2 | `Map`, `Angle` (replaces Float angles in Rotation/Arc/Canvas), `Range`, `enumerated()`, `clamped`, type deduction for templates/function expressions, unicode code points in strings, named args in any order; text modifiers (LineLimit, FontFamily, FontSize, MultilineTextAlignment, TextColor); XYPad, Stepper. **Breaking:** `load_komplete_ui()` removed (select module in Instrument Options instead); `Array.filter` → `filtered(by:)`; `subsequence(count:)` → `length:`; `String.replace_all` → `replacing_all`; `String.removing` removed; if-else expression → ternary `a ? b : c`; Ring → Arc; font size is Int; `FontFamily` (class) → `FontFamilyName`; Math `clamp`/`clampf`, `degrees_to_radians`/`radians_to_degrees` removed; `TogglePolicy` → `TriggerPolicy`; exported types must not refer to un-exported types (annotate e.g. `export var main: Component = Main()`); TapGesture cancel/up semantics changed |
| 8.2.1 | Popover fixes (initial visible, anchor following, reload crash) |
| 8.3 | `KSPTable.length` |
| 8.4 | `TextInput` component; `text` property on KSPButton/KSPKnob/KSPSwitch/KSPValueEdit; KSPTextEdit `text` setter; fixes for declarative for/if update-before-body and cycle errors |
| 8.5 | Stable `Drag`/`Drop` modifiers; enums as map keys; compile-time improvements |
| 8.5.1 | Popover `visible: Bool` constructor |
| 8.8 | `KSPMenu.entries`; `Help` modifier (Info Pane help text) |
| 8.9 | Default function arguments (`fun f(p: Int = 0)`); `\r` escape in strings |
| 8.10 | `atan2` |
| 8.11 | `letter_spacing` (Text, TextInput) + `LetterSpacing` modifier; `line_height` (Text) + `LineHeight` modifier; faster number parsing; subscript operator (`at`/`assign`) for custom classes |
| 8.12 | Method overloading; `audio_components`, `path`, and `uri` packages; Kontakt module `Instrument`/`instrument`/`Group`/`GroupList`/`GroupListIterator`/`load_sample`/`library_path`/`Sample`/`Zone`/`ZoneList`/`ZoneListIterator`; `Array.append` overloaded to accept `[Element]`, replacing `append_all` |
