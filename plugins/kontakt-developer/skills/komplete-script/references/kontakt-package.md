<!-- Condensed from Komplete UI docs (Kontakt package) -->

# `kontakt` Package Reference

Bridge between Komplete UI and Kontakt / KSP. Import everything with:

```kscript
import * from kontakt
```

## KSP Connection Model

- Each `KSP*` class mirrors a KSP UI control declaration (`ui_button`, `ui_knob`, `ui_slider`, `ui_switch`, `ui_table`, `ui_label`, `ui_level_meter`, `ui_menu`, `ui_text_edit`, `ui_value_edit`, `ui_xy`).
- Connect by constructing with `id:` — the string is the KSP control ID, i.e. the name of the variable declared in KSP (e.g. `KSPKnob(id: "my_knob")` connects to `declare ui_knob $my_knob(...)`).
- `connected: Bool` reports whether a corresponding KSP control was found for the given `id`; a control might not be connected if none was found.
- `custom_id: Int` mirrors `$CONTROL_PAR_CUSTOM_ID`; `label` mirrors `$CONTROL_PAR_LABEL`; `text` mirrors `$CONTROL_PAR_TEXT` (Kontakt 8.4+); `default_value` mirrors `$CONTROL_PAR_DEFAULT_VALUE`.
- Continuous controls (`KSPKnob`, `KSPSlider`, `KSPSwitch`, `KSPXYPadCursor`) have `automate: Bool (get/set)` — set `true` while the user interacts with the control to write automation (automation touch), set back to `false` when the interaction ends.
- Continuous value controls expose `value: Int`, plus `normalized_value: Float` (normalized using `min` and `max`) and, where applicable, `display_value: Float` (`value` scaled by `display_ratio`).

---

## KSPButton

Connects to a `ui_button` in KSP.

```kscript
import { KSPButton } from kontakt

var button = KSPButton(id: "my_button")
```

Constructors:
- `constructor(id: String)` — connects to a `ui_button` with the given control ID.

Properties:
- `checked: Bool (get/set)` — whether the button is checked.
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `id: String (get)` — the KSP control ID this button is connected to.
- `text: String (get)` — `$CONTROL_PAR_TEXT`. Kontakt 8.4+.

Methods:
- `toggle() -> ()` — toggles the checked state.

## KSPKnob

Connects to a `ui_knob` in KSP.

```kscript
import { KSPKnob } from kontakt

var knob = KSPKnob(id: "my_knob")
```

Constructors:
- `constructor(id: String)`

Properties:
- `automate: Bool (get/set)` — automation touch (see connection model).
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `default_value: Int (get)` — `$CONTROL_PAR_DEFAULT_VALUE`.
- `display_ratio: Int (get)`
- `display_value: Float (get)` — value scaled by `display_ratio`.
- `id: String (get)`
- `label: String (get)` — `$CONTROL_PAR_LABEL`.
- `max: Int (get)`
- `min: Int (get)`
- `normalized_value: Float (get/set)` — value normalized using `min` & `max`.
- `text: String (get)` — `$CONTROL_PAR_TEXT`. Kontakt 8.4+.
- `value: Int (get/set)`

## KSPSlider

Connects to a `ui_slider` in KSP.

```kscript
import { KSPSlider } from kontakt

var slider = KSPSlider(id: "my_slider")
```

Constructors:
- `constructor(id: String)`

Properties:
- `automate: Bool (get/set)`
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `default_value: Int (get)` — `$CONTROL_PAR_DEFAULT_VALUE`.
- `id: String (get)` — the KSP variable name for this control.
- `label: String (get)` — `$CONTROL_PAR_LABEL`.
- `max: Int (get)`
- `min: Int (get)`
- `normalized_value: Float (get/set)` — normalized using `min` & `max`.
- `value: Int (get/set)`

## KSPSwitch

Connects to a `ui_switch` in KSP.

```kscript
import { KSPSwitch } from kontakt

var switch = KSPSwitch(id: "my_switch")
```

Constructors:
- `constructor(id: String)`

Properties:
- `automate: Bool (get/set)`
- `checked: Bool (get/set)` — whether the switch is checked.
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `id: String (get)`
- `label: String (get)` — `$CONTROL_PAR_LABEL`.
- `text: String (get)` — `$CONTROL_PAR_TEXT`. Kontakt 8.4+.

Methods:
- `toggle() -> ()` — toggles the checked state.

## KSPTable

Connects to a `ui_table` in KSP.

```kscript
import { KSPTable } from kontakt

var table = KSPTable(id: "my_table")
```

Constructors:
- `constructor(id: String)`

Properties:
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `id: String (get)`
- `max: Int (get)`
- `min: Int (get)`
- `values: [Int] (get)` — the values of all columns.
- `length: Int (get)` — number of columns. Kontakt 8.3+.

Methods:
- `set_value(_ value: Int, at index: Int) -> ()` — sets the value of the column at the given index.
- `value(at index: Int) -> (Int)` — returns the value of the column at the given index.

## KSPLabel

Connects to a `ui_label` in KSP.

```kscript
import { KSPLabel } from kontakt

var label = KSPLabel(id: "my_label")
```

Constructors:
- `constructor(id: String)`

Properties:
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `id: String (get)`
- `text: String (get)` — the label's text value.

## KSPLevelMeter

Connects to a `ui_level_meter` in KSP.

```kscript
import { KSPLevelMeter } from kontakt

var meter = KSPLevelMeter(id: "my_level_meter")
```

Constructors:
- `constructor(id: String)`

Properties:
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `id: String (get)`
- `value: Int (get)` — the level meter's value.

## KSPMenu

Connects to a `ui_menu` in KSP.

```kscript
import { KSPMenu, KSPMenuEntry } from kontakt

var menu = KSPMenu(id: "my_menu")
```

Constructors:
- `constructor(id: String)`

Properties:
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `entries: [KSPMenuEntry] (get)` — all menu entries. Kontakt 8.8+.
- `entry_count: Int (get)` — number of entries.
- `id: String (get)` — the KSP variable name for this control.
- `selected_entry: KSPMenuEntry (get)` — currently selected entry.
- `selected_index: Int (get/set)` — currently selected entry index.
- `visible_entries: [KSPMenuEntry] (get)` — all visible entries.

## KSPMenuEntry

Describes a single menu entry.

Properties:
- `index: Int (get)` — index of this entry in the menu.
- `name: String (get)` — the name displayed.
- `value: Int (get)` — the value assigned to this entry.
- `visible: Bool (get)` — whether this entry is visible.

## KSPTextEdit

Connects to a `ui_text_edit` in KSP.

```kscript
import { KSPTextEdit } from kontakt

var text_edit = KSPTextEdit(id: "my_text_edit")
```

Constructors:
- `constructor(id: String)`

Properties:
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `id: String (get)`
- `text: String (get/set)` — the TextEdit's value. Setter Kontakt 8.4+.

## KSPValueEdit

Connects to a `ui_value_edit` in KSP.

```kscript
import { KSPValueEdit } from kontakt

var value_edit = KSPValueEdit(id: "my_value_edit")
```

Constructors:
- `constructor(id: String)`

Properties:
- `connected: Bool (get)`
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `display_ratio: Int (get)`
- `display_value: Float (get)` — value scaled by `display_ratio`.
- `id: String (get)`
- `max: Int (get)`
- `min: Int (get)`
- `normalized_value: Float (get/set)` — normalized using `min` & `max`.
- `text: String (get)` — `$CONTROL_PAR_TEXT`. Kontakt 8.4+.
- `value: Int (get/set)`

## KSPXYPad

Connects to a `ui_xy` in KSP.

```kscript
import { KSPXYPad, KSPXYPadCursor } from kontakt

var xy_pad = KSPXYPad(id: "my_xy")
```

Constructors:
- `constructor(id: String)`

Properties:
- `active_index: Int (get)` — the actively controlled index.
- `connected: Bool (get)`
- `cursor_count: Int (get)` — number of cursors.
- `cursors: [KSPXYPadCursor] (get)` — all cursors.
- `custom_id: Int (get)` — `$CONTROL_PAR_CUSTOM_ID`.
- `id: Int (get)` — the KSP variable name for this control. (Documented type is `Int`.)
- `visible_cursors: [KSPXYPadCursor] (get)` — all visible cursors.

Methods:
- `cursor(at index: Int) -> (KSPXYPadCursor)` — returns the cursor at the given index. The index must be a multiple of two: 0 is the first cursor, 2 is the second.

## KSPXYPadCursor

Controls a single cursor in an XY pad.

Properties:
- `automate: Bool (get/set)` — automation touch.
- `automation_name: String (get)` — the cursor's automation name (`$CONTROL_PAR_AUTOMATION_NAME`).
- `visible: Bool (get)` — whether the cursor is visible to the user.
- `x: Float (get/set)` — x position.
- `y: Float (get/set)` — y position.

---

## Help Modifier (Kontakt 8.8+)

`modifier Help` — displays help text in Kontakt's Info Pane when hovering over the component it is applied to.

Constructor:
- `(_ text: String)` — the text to display in the Info Pane.

```kscript
import { Rectangle } from ui
import { Help } from kontakt

export var main = Rectangle(color: Color(0xFF242424)) with {
    Help("Info Text")
}
```

- External text: an optional `info_hints.json` in `Resources/komplete_scripts` maps the modifier text (used as key) to the actual help text; if the key is missing, the key itself is displayed. A malformed `info_hints.json` logs an error and all `Help` modifiers display their literal text.
- Rich text: supports `<b>Bold</b>`, `<i>Italic</i>`, `<u>Underlined</u>` in both the modifier text and `info_hints.json` values.
- Layout behavior: none.

---

## Instrument (Kontakt 8.12)

Represents a Kontakt instrument and provides access to its groups and zones. Cannot be instantiated; the current instrument is available via the `instrument` variable:

```kscript
import { instrument } from kontakt
```

```kscript
export class Instrument {
    groups: GroupList { get }

    group(named: String) -> Group?
    zone(id: Int) -> Zone?
}

export var instrument: Instrument
```

Properties:
- `groups: GroupList (get)` — all groups in the instrument.

Methods:
- `group(named: String) -> (Group?)` — returns the first group with the given name, or `nil` if no such group exists.
- `zone(id: Int) -> (Zone?)` — returns the zone with the given ID, or `nil` if no such zone exists. Mainly for interoperability with KSP, e.g. resolving a zone ID persisted via KSP back to a `Zone`.

## Group (Kontakt 8.12)

Represents a group in a Kontakt instrument and provides access to its zones. Cannot be instantiated directly; returned by `Instrument.groups`, `Instrument.group`, `Zone.group`.

```kscript
import { Group } from kontakt
```

Properties:
- `name: String (get)` — the name of the group.
- `zones: ZoneList (get)` — all zones in the group.

Methods:
- `add_zone(_ sample: Sample) -> (Zone)` — creates a new zone in this group from the given sample and returns it. Zones created this way are destroyable via `Zone.destroy`.
- `to_string() -> (String)` — string representation of the group.

## GroupList (Kontakt 8.12)

Ordered collection of `Group` objects. Cannot be instantiated directly; returned by `Instrument.groups`.

```kscript
import { GroupList } from kontakt
```

Properties:
- `length: Int (get)` — number of groups in the list.

Methods:
- `at(index: Int) -> (Group)` [subscript] — returns the group at the given index. Throws an error if out of range. Also enables the subscript operator; equivalent forms:

```kscript
var group = group_list.at(index: 0)
var group = group_list[0]
```

- `iterator() -> (GroupListIterator)` — returns an iterator; mainly exists to enable `for-in` syntax:

```kscript
for group in group_list {
    // ...
}
```

## GroupListIterator (Kontakt 8.12)

Sequential iteration over a `GroupList`. Cannot be instantiated directly; returned by `GroupList.iterator`.

```kscript
import { GroupListIterator } from kontakt
```

Properties:
- `has_next: Bool (get)` — whether the iterator has more groups to yield.

Methods:
- `next() -> (Group)` — advances the iterator and returns the next group.

## Zone (Kontakt 8.12)

Represents a zone in a Kontakt instrument. Cannot be instantiated directly; returned by `Group.add_zone`, `Instrument.zone`, `ZoneList.at`.

```kscript
import { Zone } from kontakt
```

Properties:
- `id: Int (get)` — the unique identifier of the zone. Can be passed to KSP for persistence and resolved back to a `Zone` via `Instrument.zone`.
- `group: Group (get)` — the group this zone belongs to.
- `is_destroyable: Bool (get)` — whether the zone can be destroyed via `destroy`. Only zones created by the script are destroyable; pre-existing zones and zones owned by KSP are not.
- `sample: Sample? (get)` — the audio sample assigned to this zone, or `nil` if the zone has no sample.

Methods:
- `destroy() -> ()` — destroys this zone, removing it from the instrument. Destroying an already destroyed zone is a no-op. Throws an error if the zone is not destroyable (see `is_destroyable`).
- `to_string() -> (String)` — string representation of the zone.

## ZoneList (Kontakt 8.12)

Ordered collection of `Zone` objects. Cannot be instantiated directly; returned by `Group.zones`.

```kscript
import { ZoneList } from kontakt
```

Properties:
- `length: Int (get)` — number of zones in the list.

Methods:
- `at(index: Int) -> (Zone)` [subscript] — returns the zone at the given index. Also enables the subscript operator; equivalent forms:

```kscript
var zone = zone_list.at(index: 0)
var zone = zone_list[0]
```

- `iterator() -> (ZoneListIterator)` — returns an iterator; mainly exists to enable `for-in` syntax:

```kscript
for zone in zone_list {
    // ...
}
```

## ZoneListIterator (Kontakt 8.12)

Sequential iteration over a `ZoneList`. Cannot be instantiated directly; returned by `ZoneList.iterator`.

```kscript
import { ZoneListIterator } from kontakt
```

Properties:
- `has_next: Bool (get)` — whether the iterator has more zones to yield.

Methods:
- `next() -> (Zone)` — advances the iterator and returns the next zone.

## Sample (Kontakt 8.12)

Represents an audio sample loaded in Kontakt. Cannot be instantiated directly; returned by `load_sample` or by querying a sample from an existing zone.

```kscript
import { Sample } from kontakt
```

Properties:
- `num_channels: Int (get)` — number of channels.
- `num_frames: Int (get)` — number of frames (samples per channel).
- `path: Path (get)` — file path of the audio sample. Type changed from `String` to `Path` in Kontakt 8.12.
- `sample_rate: Int (get)` — sample rate in Hz.

Methods:
- `to_string() -> (String)` — string representation of the sample.

## Utilities (free functions / variables)

- `create_tmp_midi_file(for_export_area: Int) -> (String)` — creates a temporary MIDI file for the given export area and returns a file URL to it. Use to enable MIDI drag and drop.
- `library_path: Path` — the path of the Kontakt library. Kontakt 8.12.
- `load_sample(path: Path) -> (Sample?)` — loads an audio sample from the given file path; returns the existing sample if one with this path is already loaded. Returns nil if the sample cannot be loaded. Kontakt 8.12.
