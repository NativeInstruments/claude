<!-- Condensed from Komplete UI docs (Kontakt Controls package) -->
# Kontakt Controls Package

Ready-made UI controls that connect to KSP ui controls via `control_id`. **Not included by default**: download the package manually (https://storage.googleapis.com/ni-developer-platform/kontakt_controls.zip) and copy its folder into the instrument's `Resources/komplete_scripts/` folder, then:

Components: `Slider`, `Knob`, `XYPad`, `Stepper`, `Switch`, `ToggleButton`. Only a small set of common controls is covered; it is extended over time.

## Slider (KSP `ui_slider`)

```kscript
import { Slider } from kontakt_controls

export var main = Slider(
    control_id: "my_slider",
    label: "My Slider",
)
```

Constructor:
- `control_id: String` (required) — name of the KSP control
- `label: String` (required) — text shown while resting
- `step_count: Int? = nil` — steps from KSP min to max; `nil` = step size 1. Step counts > max-min are clipped; non-integer step sizes are adjusted to nearest integer step size.
- `fine_step_count: Int = 1` — sub-steps per step while holding Shift (finetuning)
- `style: SliderStyle = default_slider_style`
- `label_value_source: LabelValueSource = LabelValueSource.ksp_control_value`
- `value_to_string: ValueFormatter = default_value_formatter` — formats value shown during interaction; ignored when `label_value_source` is `.ksp_label_property`
- `sensitivity: Float = 1` — 1 = full range over one edge-to-edge drag
- `disabled: Bool = false`

Behaviours: Shift-drag = finetuning; Ctrl (Win) / Cmd (macOS) + click = reset to default value.

```kscript
export type SliderStyle = DraggableControlStyle
export var default_slider_style = SliderStyle(
    width: nil,
    height: nil,
    image: default_slider_image,
    handle_size: 16,
    padding: EdgeInsets(0),
    label_style: LabelStyle(),
    axis: Axis.horizontal,
    disabled_opacity: 0.24,
)
```

KSP commonly uses `ui_slider` for controls that look like knobs — pass `style: default_knob_style` to a `Slider` for that.

## Knob (KSP `ui_knob`)

```kscript
import { Knob } from kontakt_controls

export var main = Knob(
    control_id: "my_knob",
    label: "My Knob",
)
```

Constructor: identical parameters/semantics to `Slider`, except `sensitivity: Float = 0.5` and `style: KnobStyle = default_knob_style`. Same Shift-finetuning and Ctrl/Cmd-tap reset behaviours.

```kscript
export type KnobStyle = DraggableControlStyle
export var default_knob_style = KnobStyle(
    width: nil,
    height: nil,
    image: default_knob_image,
    handle_size: 0,
    padding: EdgeInsets(0),
    label_style: LabelStyle(),
    axis: Axis.vertical,
    disabled_opacity: 0.24,
)
```

## XYPad (KSP `ui_xy`)

Controls the active cursor of a KSP `ui_xy`.

```kscript
import { XYPad } from kontakt_controls

export var main = XYPad(
    control_id: "my_xy",
    label: "XY Pad",
)
```

```kscript
export component XYPad {
    @property control_id: String
    @property label: String
    @property step_count_x: Int? = nil
    @property fine_step_count_x: Int = 2
    @property sensitivity_x: Float = 1
    @property step_count_y: Int? = nil
    @property fine_step_count_y: Int = 2
    @property sensitivity_y: Float = 1
    @property value_to_string: XYValueFormatter = default_xy_value_formatter
    @property style: XYPadStyle = default_xy_pad_style
    @property disabled: Bool = false
}
```

- `step_count_x/y: nil` = finest resolution; `fine_step_count_x/y` subdivides steps on Shift (1 disables fine tuning; with `step_count` nil it reduces sensitivity)
- `value_to_string` default shows raw KSP values rounded to two digits

```kscript
export class XYPadStyle {
    width: Float? = nil                 // nil = image size
    height: Float? = nil                // nil = image size
    background_image: ImageAsset = default_xy_background_image
    handle_size: Float? = nil           // nil = handle_image size; square
    handle_image: ImageAsset = default_xy_handle_image
    inner_padding: EdgeInsets = EdgeInsets(3)  // shrinks interactive area (image border)
    label_style: LabelStyle = default_xy_pad_label_style
    disabled_opacity: Float = 0.24

    copy() -> (XYPadStyle)
    deep_copy() -> (XYPadStyle)
    overriding(_ modify: (XYPadStyle) -> ()) -> (XYPadStyle)
}
```

## Stepper (KSP `ui_value_edit`)

```kscript
import { Stepper } from kontakt_controls

export var main = Stepper(
    control_id: "my_value_edit",
    label: "My Value Edit",
    default_value: 50,
)
```

```kscript
export component Stepper {
    @property control_id: String
    @property label: String
    @property default_value: Int      // reset target for CTRL/CMD+Tap (temp, until fetchable from ui_value_edit)
    @property text_previous: String? = nil
    @property text_next: String? = nil
    @property step_count: Int? = nil  // nil = finest resolution
    @property fine_step_count: Int = 2
    @property sensitivity: Float = 0.1
    @property style: StepperStyle = StepperStyle()
    @property value_to_string: ValueFormatter = default_value_formatter
    @property disabled: Bool = false
    @property on_editing_changed: (Bool) -> () = fun (editing: Bool) {}
    @property policy: TriggerPolicy = TriggerPolicy.on_down
}
```

```kscript
export class StepperStyle {
    width: Float? = nil
    height: Float? = nil
    buttons_position: ButtonsPosition = ButtonsPosition.left_right
    spacing: Float = 2                 // space between arrows and value display
    buttons_style: PrevNextButtonsStyle = default_previous_next_buttons_style
    value_padding: EdgeInsets = EdgeInsets(5)
    value_style: LabelStyle = default_value_style
    label_style: LabelStyle = default_label_style
    disabled_opacity: Float = 0.24

    copy() -> (StepperStyle)
    deep_copy() -> (StepperStyle)
    overriding(_ modify: (StepperStyle) -> ()) -> (StepperStyle)
}

export class PrevNextButtonsStyle {
    previous_style: ButtonStyle = default_button_style
    next_style: ButtonStyle = default_button_style
    axis: Axis = Axis.horizontal
    spacing: Float = 5

    copy() -> (PrevNextButtonsStyle)
    deep_copy() -> (PrevNextButtonsStyle)
    overriding(_ modify: (PrevNextButtonsStyle) -> ()) -> (PrevNextButtonsStyle)
}

export class ButtonStyle {
    image: ImageAsset? = nil
    width: Float? = nil
    height: Float? = nil
    padding: EdgeInsets = EdgeInsets(0)
    font_family: ui.FontFamilyName = default_font
    font_weight: Int = ui.font_weights.normal
    italic: Bool = false
    text_size: Int = 12
    text_colors: ButtonStateColors = ButtonStateColors(
        resting: Color(0x99EEEEEE),
        hovered: Color(0xBBEEEEEE),
        pressed: Color(0xFFEEEEEE),
        disabled: Color(0x3DEEEEEE),
    )
    text_alignment: ui.Alignment = ui.Alignment.center

    copy() -> (ButtonStyle)
    deep_copy() -> (ButtonStyle)
    overriding(_ modify: (ButtonStyle) -> ()) -> (ButtonStyle)
}

export class ButtonStateColors {
    resting: Color
    hovered: Color
    pressed: Color
    disabled: Color

    copy() -> (ButtonStateColors)
    overriding(_ modify: (ButtonStateColors)->()) -> (ButtonStateColors)
}

export enum ButtonsPosition {
    left,
    right,
    left_right,
    top_bottom
}
```

## Switch (KSP `ui_switch`)

```kscript
import { Switch } from kontakt_controls

export var main = Switch(control_id: "my_switch")
```

Constructor:
- `control_id: String` (required)
- `text: String? = nil`
- `style: SwitchStyle = default_switch_style`
- `disabled: Bool = false`
- `policy: TriggerPolicy = TriggerPolicy.on_down`

```kscript
export type SwitchStyle = ToggleStyle
export type SwitchStateColors = ToggleStateColors

export var default_switch_style = ToggleStyle(
    image: default_switch_image,
    width: nil,
    height: nil,
    padding: EdgeInsets(0),
    font_family: default_font,
    font_weight: ui.font_weights.normal,
    italic: false,
    text_size: 12,
    text_colors: SwitchStateColors(
        resting: Color(0xFFEEEEEE),
        hovered: Color(0xFFEEEEEE),
        pressed: Color(0xFFEEEEEE),
        checked: Color(0xFFEEEEEE),
        checked_and_hovered: Color(0xFFEEEEEE),
        checked_and_pressed: Color(0xFFEEEEEE),
        disabled: Color(0x3DEEEEEE),
        checked_and_disabled: Color(0x3DEEEEEE),
    ),
    text_alignment: ui.Alignment.center,
)
```

## ToggleButton (KSP `ui_button`)

```kscript
import { ToggleButton } from kontakt_controls

export var main = ToggleButton(control_id: "my_button")
```

Constructor: same as `Switch`, except `style: ToggleButtonStyle = default_toggle_button_style` and `policy: TriggerPolicy = TriggerPolicy.on_up`.

```kscript
export type ToggleButtonStyle = ToggleStyle
export var default_toggle_button_style = ToggleButtonStyle(
    image: default_toggle_button_image,
    width: nil,
    height: nil,
    padding: EdgeInsets(horizontal: 18, vertical: 4),
    font_family: default_font,
    font_weight: ui.font_weights.normal,
    italic: false,
    text_size: 12,
    text_colors: ToggleStateColors(
        resting: Color(0xFFEEEEEE),
        hovered: Color(0xFFEEEEEE),
        pressed: Color(0xFFEEEEEE),
        checked: Color(0xFFEEEEEE),
        checked_and_hovered: Color(0xFFEEEEEE),
        checked_and_pressed: Color(0xFFEEEEEE),
        disabled: Color(0x3DEEEEEE),
        checked_and_disabled: Color(0x3DEEEEEE),
    ),
    text_alignment: ui.Alignment.center,
)
```

`ToggleStyle` is shared between `Switch` and `ToggleButton` — their default styles are interchangeable.

## Style Classes

### DraggableControlStyle (used by Slider/Knob)

```kscript
export class DraggableControlStyle {
    width: Float? = nil                 // nil = image width
    height: Float? = nil                // nil = image height
    image: ImageAsset = default_knob_image
    handle_size: Float = 0.0            // handle size along the drag axis
    padding: EdgeInsets = EdgeInsets(0.0)  // extra interactive area around the image
    label_style: LabelStyle = LabelStyle()
    axis: Axis = Axis.vertical
    disabled_opacity: Float = 0.24

    copy() -> (DraggableControlStyle)
    deep_copy() -> (DraggableControlStyle)
    overriding(_ modify: (DraggableControlStyle) -> ()) -> (DraggableControlStyle)
}
```

```kscript
var modified = (DraggableControlStyle()).overriding(fun (style) {
    style.axis = Axis.horizontal
})
```

### ToggleStyle (used by Switch/ToggleButton)

Two modes: **text mode** (text sets the size; image is stretched behind text plus padding) and **image mode** (no text; image plus padding sets the size).

Properties:
- `image: ImageAsset? = default_toggle_button_image`
- `width: Float? = nil` / `height: Float? = nil` — override default size (text- or image-based)
- `padding: EdgeInsets = EdgeInsets(horizontal: 18, vertical: 4)`
- `font_family: FontFamilyName = default_font`
- `font_weight: Int = ui.font_weights.normal`
- `italic: Bool = false`
- `text_size: Int = 12`
- `text_colors: ToggleStateColors` — defaults: resting/hovered/pressed/checked/checked_and_hovered/checked_and_pressed `Color(0xFFEEEEEE)`, disabled/checked_and_disabled `Color(0x3DEEEEEE)`
- `text_alignment: ui.Alignment = ui.Alignment.center` — only relevant if width/height larger than text

Methods: `copy()`, `deep_copy()` (copies `padding` and `text_colors`), `overriding(_ modify: (ToggleStyle) -> ()) -> (ToggleStyle)`.

### ToggleStateColors

```kscript
export class ToggleStateColors {
    resting: Color
    hovered: Color
    pressed: Color
    checked: Color
    checked_and_hovered: Color
    checked_and_pressed: Color
    disabled: Color
    checked_and_disabled: Color

    copy() -> (ToggleStateColors)
    deep_copy() -> (ToggleStateColors)
    overriding(_ modify: (ToggleStateColors) -> ()) -> (ToggleStateColors)
}
```

### LabelStyle

```kscript
export class LabelStyle {
    color: Color = Color(0xFFEEEEEE)
    size: Int = 12
    font: ui.FontFamilyName = default_font
    font_weight: Int = ui.font_weights.normal
    italic: Bool = false
    max_width: Float = 100
    alignment: ui.Alignment = ui.Alignment.center
    offset_x: Float = 0
    offset_y: Float = 30

    copy() -> (LabelStyle)
    overriding(_ modify: (LabelStyle) -> ()) -> (LabelStyle)
}
```

### EdgeInsets

```kscript
export class EdgeInsets {
    left: Float
    right: Float
    top: Float
    bottom: Float
    horizontal: Float { get, set }   // set: sets left+right; get: sum of left+right
    vertical: Float { get, set }     // set: sets top+bottom; get: sum of top+bottom

    constructor(_ length: Float)     // same inset on all sides
    constructor(horizontal: Float = 0.0, vertical: Float = 0.0)
    constructor(left: Float = 0.0, right: Float = 0.0, top: Float = 0.0, bottom: Float = 0.0)

    copy() -> (EdgeInsets)
    overriding(_ modify: (EdgeInsets) -> ()) -> (EdgeInsets)
}
```

### ImageAsset

Image with metadata for controls. Supports sprites (`frame_count`) and nine-patch (`fixed_*` borders).

```kscript
import { ImageAsset } from kontakt_controls

var my_image = ImageAsset(
    path: "rounded_button.png",   // project-relative path
    width: 128,                   // logical pixels
    height: 24,
    frame_count: 8,               // default 1 (non-sprite)
    fixed_left: 4,                // nine-patch fixed borders, default 0
    fixed_right: 4,
    fixed_top: 4,
    fixed_bottom: 4,
)
```

## Enums

```kscript
export enum Axis {
    horizontal,
    vertical
}

export enum LabelValueSource {
    // Use the value of the KSP control for the label
    ksp_control_value,
    // Use the KSP label property (CONTROL_PAR_LABEL)
    ksp_label_property,
}

export enum TriggerPolicy {
    // Toggles on pointer down
    on_down,
    // Toggles on pointer up
    on_up,
}
```

## Types and Formatters

```kscript
export type ValueFormatter = (/* value */Int, /* min */Int, /* max */Int) -> (String)
export type XYValueFormatter = (/* cursor position */Point) -> (String)

// Converts value to String, e.g. 5 -> "5"
export fun default_value_formatter(_ value: Int, min: Int, max: Int) -> (String)
// Converts ui_xy cursor to "x.xx // y.yy", e.g. "0.12 // 0.23"
export fun default_xy_value_formatter(_ cursor: Point) -> (String)
```

## Constants

```kscript
import { default_font } from kontakt_controls  // font used by all default styles

import {
    default_toggle_button_image,
    default_switch_image,
    default_knob_image,
    default_slider_image,
    default_xy_handle_image,
    default_xy_background_image,
}
from kontakt_controls
```

## Utility Functions

```kscript
// Maps the integer value to the range [0, 1]
normalize(_ value: Int, min: Int, max: Int) -> (Float)
// Maps the normalized value to the closest integer value in [min, max]
denormalize(_ value: Float, min: Int, max: Int) -> (Int)
```

## How-To: Custom Styles for Components

Define styles in a separate file (e.g. `Resources/komplete_scripts/style/my_styles.kscript`), start from a default style, and apply via the `style:` parameter:

```kscript
// my_styles.kscript
import { default_toggle_button_style, ImageAsset, EdgeInsets, ToggleStateColors } from kontakt_controls
import { Alignment, font_weights } from ui

// Sprite must contain 8 frames mapped to ToggleState order:
// resting, hovered, pressed, checked, checked_and_hovered,
// checked_and_pressed, disabled, checked_and_disabled
// (frames may repeat; provide @2x variant alongside, e.g. custom_toggle_button_@2x.png)
var custom_toggle_button_image = ImageAsset(
    path: "assets/images/custom_toggle_button.png",
    width: 72,
    height: 24,
    frame_count: 8,
    fixed_left: 12,
    fixed_right: 12,
    fixed_top: 12,
    fixed_bottom: 12,
)

export var custom_toggle_button_style =
    default_toggle_button_style.overriding(fun(style) {
        style.image = custom_toggle_button_image
        style.width = 100            // fixed size; or omit and use padding
        style.height = 40            // for text-driven size
        style.padding = EdgeInsets(4)  // keeps text off borders; text elides when too long
        style.font_weight = font_weights.bold
        style.italic = true
        style.text_size = 20
        style.text_colors = ToggleStateColors(
            resting: Color(0xFFC780FF),
            hovered: Color(0xFFB24DFF),
            pressed: Color(0xFFB24DFF),
            checked: Color(0xFFB24DFF),
            checked_and_hovered: Color(0xFFB24DFF),
            checked_and_pressed: Color(0xFFB24DFF),
            disabled: Color(0xFFC780FF),
            checked_and_disabled: Color(0xFFC780FF),
        )
        style.text_alignment = Alignment.center_left
})
```

```kscript
// main.kscript
import * from ui
import * from kontakt_controls
import { custom_toggle_button_style } from style.my_styles

component Main {
    ZStack {
        Rectangle(color: Color(0xFF2A2A2A))
        ToggleButton(
            control_id: "my_button",
            text: "Hello",
            style: custom_toggle_button_style
        )
    }
}

export var main: Component = Main()
```

To style from scratch instead of overriding defaults, call the style constructor directly (e.g. `ToggleButtonStyle(...)`). For a custom font, load it first and set `style.font_family = custom_font`. With text mode, the background is a NinePatchImage, so fixed `width`/`height` work; without padding text can touch/overflow borders.

Corresponding KSP (declare + expose):

```
on init
   declare ui_button $my_button
   make_persistent($my_button)
   read_persistent_var($my_button)
   expose_controls
end on
```

## How-To: Create Your Own Components

Build vector-based (resizable, colorable) controls from UI-package shapes plus the package's `Draggable` modifier. Pattern for a custom knob:

1. Appearance from shapes (e.g. `Arc` from `ui`).
2. Connect to KSP via a computed state returning a KSP control wrapper (`KSPKnob(id:)` from `kontakt`); read `value`, `min`, `max`, `normalized_value`, `default_value`.
3. Interaction via `Draggable` (drag gesture + Ctrl/Cmd+tap reset), implementing `on_editing_changed` and `on_reset`.

```kscript
// my_knob.kscript
import * from ui
import { KSPKnob } from kontakt
import { EdgeInsets, Draggable, DraggableAxis } from kontakt_controls

export component MyKnob {
    @property control_id: String
    @property disabled: Bool = false

    ksp_control: KSPKnob { return KSPKnob(id: self.control_id) }

    // binding target for the unused horizontal axis
    noop: Float {
        get { return 0 }
        set(new_value) {}
    }

    axis: DraggableAxis {
        return DraggableAxis(
            step_count: self.ksp_control.max - self.ksp_control.min,
            fine_step_count: 2,
            sensitivity: 0.5,
        )
    }

    Arc(
        color: Color(0xFFC780FF),
        angle: Angle(degrees: self.ksp_control.normalized_value*360),
        start_angle: Angle(degrees: 90),
    ) with {
        Frame (width: 60, height: 60)
        Overlay {
            Text("\{self.ksp_control.value}", color: Color(0xFFC780FF))
        }
        Draggable(
            x_value: self.$noop,
            y_value: self.ksp_control.$normalized_value,
            y_axis: self.axis,
            disabled: self.disabled,
            on_editing_changed: fun (editing: Bool) {
                self.ksp_control.automate = editing
            },
            on_reset: fun () {
                self.ksp_control.value = self.ksp_control.default_value
            },
            inset: EdgeInsets(0)
        )
    }
}
```

Usage: `import { MyKnob } from my_knob` then `MyKnob(control_id: "my_knob")`. The same `Draggable`-based approach works for custom sliders; Switch/ToggleButton analogues follow the same pattern. Matching KSP:

```
on init
   declare ui_knob $my_knob (0, 1000, 1)
   make_persistent($my_knob)
   read_persistent_var($my_knob)
   expose_controls
end on
```
