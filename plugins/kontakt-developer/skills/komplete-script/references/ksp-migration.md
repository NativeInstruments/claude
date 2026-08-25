<!-- Condensed from Komplete UI docs (KSP migration) -->

# KSP → Komplete UI Migration

Side-by-side KSP and kscript for developers coming from the Kontakt Script Processor.
Language details are in `kscript-developer:komplete-script` → `references/language.md`.
Komplete Script mixes imperative code (logic: variables, functions, loops) and declarative code (UI: components, modifiers, layout). Unlike KSP's global step-by-step style, the UI is described as what it should look like given state (reactivity) — no manual update logic.

## Variables

Declared anywhere in imperative scope, must be initialized (no defaults), type usually inferred:

```txt
declare $test := -1
```
```kscript
var test_implicit = -1
var test_explicit: Int = -1 // optional explicit type
```

## Arrays and Maps

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

## Functions

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

## Entry point

No `on init`. The main module (linked in Instrument Options) runs once at instrument load — its top-level scope is the `on init` equivalent. The main module MUST export a UI component:

```kscript
import { Text } from ui
export var main = Text("Hello World") // required in the main module
```

## Project structure

Any number of modules (files); no script-slot limit. All code goes in the instrument's Resource Container under the `komplete_scripts` folder; import modules from there.

## UI elements

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

## Callbacks and reactivity

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

## Layout

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

