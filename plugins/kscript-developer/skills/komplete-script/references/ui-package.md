<!-- Condensed from Komplete UI docs (UI package) -->
# UI Package

Fundamental components, modifiers, types and utilities for building UIs.

```kscript
import * from ui
// or selective: import { Text, HStack, Frame } from ui
```

## Components

### Arc
`Arc(color: Color, angle: Angle = Angle(degrees: 360.0), start_angle: Angle = Angle(degrees: 0.0), thickness: Float = 4.0)`
Displays a colored arc; positive angles rotate clockwise, zero degrees at 3 o'clock. Layout: fills the smaller dimension of the available area (preserves aspect ratio); clips thickness if area too small.
```kscript
import { Arc } from ui
export var main = Arc(color: Color(0xFFFF5F52), angle: Angle(degrees: 270), thickness: 9)
```

### Canvas
`Canvas(background_color: Color = Color(0x00000000), opaque: Bool = false, paint: fun (Painter, Rect))`
Custom drawing; `paint` is called whenever repaint is needed (auto re-invoked when state/properties used in it change); second arg is the canvas bounding box. Set `opaque: true` to skip blending for speed. Heavy use can hurt performance. Layout: always fills the available area.
```kscript
import { Canvas, Painter, Rect } from ui
export var main = Canvas(
    paint: fun (painter: Painter, frame: Rect) {
        painter.set_fill(Color(0xFF20639B))
        painter.draw_rect(frame)
    }
)
```

Painter methods (see Classes below for Path/Point/Rect):
- `draw_arc(center: Point, radius: Float, start_angle: Angle, end_angle: Angle)` — positive angle draws counter-clockwise
- `draw_circle(center: Point, radius: Float)`
- `draw_ellipse(_ frame: Rect)`
- `draw_line(from: Point, to: Point)`
- `draw_path(_ path: Path)`
- `draw_pie(frame: Rect, start_angle: Angle, end_angle: Angle)` — positive angle draws counter-clockwise
- `draw_rect(_ frame: Rect)`
- `save()` / `restore()` — push/pop painter state
- `rotate(_ angle: Angle)` — rotates coordinate system clockwise
- `scale(x: Float, y: Float)`, `shear(h: Float, v: Float)`, `translate(x: Float, y: Float)`
- `set_dash_offset(_ offset: Float)`
- `set_dash_pattern(_ pattern: [Float])` — even count of positive numbers; even entries = dashes, odd = spaces; units of line width; empty pattern = solid
- `set_fill(_ color: Color?)` — nil disables fills
- `set_stroke(_ color: Color?)` — nil disables strokes
- `set_line_cap(_ cap: CapStyle)`, `set_line_join(_ join: JoinStyle)`
- `set_line_width(_ width: Float)`
- `set_opacity(_ value: Float)` — 0.0 to 1.0

```kscript
enum CapStyle { flat, round, square }
enum JoinStyle { round, bevel, miter }
```
CapStyle: `flat` = square line not covering end point; `round` = rounded; `square` = square line extending beyond end point by half line width.

### Empty
`Empty()`
Renders nothing.
```kscript
import { Empty } from ui
export var main = Empty()
```

### HStack
`HStack(spacing: Float = 0, alignment: VerticalAlignment = VerticalAlignment.center, children: Template())`
Arranges children in a horizontal line; children can be a trailing block. Layout: width = sum of children widths + spacing; height = max child height.
```kscript
import { HStack, Rectangle, Frame } from ui
export var main = HStack(spacing: 10) {
    Rectangle(color: Color(0xFF8E0000)) with { Frame(width: 24, height: 48) }
    Rectangle(color: Color(0xFFC62828)) with { Frame(height: 24) }
}
```

### Image
`Image(_ path: String, frame: Int = 0, frame_count: Int = 1, resizable: Bool = false)`
Displays an image or sprite (path relative to main file); `frame`/`frame_count` select sprite frames; `resizable` allows stretching into available area (does not preserve aspect ratio). Formats: PNG, JPEG, WEBP, SVG (static SVG 1.2 Tiny), TIFF. High-DPI raster variants auto-detected via `@2x` postfix (e.g. `myimage@2x.png`). Layout: logical image size unless resizable.
```kscript
import { Image } from ui
export var main = Image("my_background.png")
```

### NinePatchImage
`NinePatchImage(_ path: String, frame: Int = 0, frame_count: Int = 1, fixed_left: Float = 0, fixed_right: Float = 0, fixed_top: Float = 0, fixed_bottom: Float = 0)`
Image with fixed corner regions and stretchable edges/center (corners scale by aspect ratio, top/bottom edges stretch horizontally, left/right edges vertically, center both). Same file formats as Image. Layout: always stretches into available area.
```kscript
import { NinePatchImage } from ui
export var main = NinePatchImage("my_nine_patch.png", fixed_left: 4, fixed_right: 4, fixed_top: 4, fixed_bottom: 4)
```

### Rectangle
`Rectangle(color: Color, radius: Float = 0.0)`
Displays a filled rectangle with optional rounded corners. Layout: fills the available area.
```kscript
import { Rectangle } from ui
export var main = Rectangle(color: Color(0xFFFF0000))
```

### Spacer
`Spacer(min_length: Float = 4.0)`
Expands along the main axis of a containing HStack/VStack (both axes otherwise); lower priority than all other components, so it takes only remaining space. Known issue: incorrect size when applying modifiers to a Spacer.
```kscript
import { Spacer, HStack, Rectangle, Frame } from ui
export var main = HStack {
    Rectangle(color: Color(0xFF8E0000)) with { Frame(width: 24, height: 48) }
    Spacer()
    Rectangle(color: Color(0xFFFF5F52)) with { Frame(width: 24, height: 48) }
} with { Frame(width: 128, height: 48) }
```

### Text
`Text(_ text: String, color: Color? = nil, size: Int? = nil, font_family: FontFamilyName? = nil, italic: Bool = false, font_weight: Int = font_weights.normal, letter_spacing: Float? = nil, line_height: Float? = nil, line_limit: Int? = nil, multiline_alignment: HorizontalAlignment? = nil)`
Displays text. `nil` params inherit from environment; root env defaults: color black, size 16, font Roboto, letter spacing 0, line height multiplier 1, line limit max_integer, multiline alignment center. `letter_spacing` (px, can be negative) and `line_height` (multiplier, e.g. 1.5 = 150%) introduced in kscript 1.8. Layout: exact space needed; wraps vertically if too narrow; elides as last resort; never uses more space than needed.
```kscript
import { Text } from ui
export var main = Text("Hello, world!")
```

### TextInput
Single line of editable text (introduced in kscript 1.1). Only offers text, cursor, selection — style it with other modifiers/components. Tapping in acquires keyboard focus, tapping outside removes it. Layout: fills available horizontal space; fixed height from font family & size.

Constructor with binding (state reflects edits immediately):
```kscript
TextInput(
    _ text: $<String>,
    read_only: Bool = false,
    color: Color? = nil,               // env default: black
    selection_color: Color? = nil,     // env default: gray
    selected_text_color: Color? = nil, // env default: black
    size: Int? = nil,                  // env default: 16
    font_family: FontFamilyName? = nil,// env default: Roboto
    italic: Bool = false,
    font_weight: Int = font_weights.normal,
    alignment: HorizontalAlignment = HorizontalAlignment.left,
    letter_spacing: Float? = nil,      // env default: 0; kscript 1.8
    on_focus_changed: (Bool) -> () = fun (focused) {},
    on_editing_finished: () -> () = fun () {},   // focus lost or enter/return
    on_submitted: () -> () = fun () {},          // enter/return
)
```

Constructor without binding (defer state updates, e.g. until submit):
```kscript
TextInput(
    _ text: String,
    // ... same styling params as above ...
    on_focus_changed: (Bool) -> () = fun (focused) {},
    on_editing_finished: (String) -> () = fun (text) {},
    on_submitted: (String) -> () = fun (text) {},
    on_text_edited: (String) -> () = fun (text) {},  // every edit (type/delete/paste)
)
```

```kscript
import { TextInput } from ui
export component Main {
    text: String = "Edit me"
    TextInput(self.$text)
}
export var main = Main()
```

### VStack
`VStack(spacing: Float = 0, alignment: HorizontalAlignment = HorizontalAlignment.center, children: Template())`
Arranges children in a vertical line; children can be a trailing block. Layout: height = sum of children heights + spacing; width = max child width.
```kscript
import { VStack, Rectangle, Frame } from ui
export var main = VStack(spacing: 10) {
    Rectangle(color: Color(0xFF8E0000)) with { Frame(width: 48, height: 24) }
    Rectangle(color: Color(0xFFC62828)) with { Frame(width: 24) }
}
```

### ZStack
`ZStack(alignment: Alignment = Alignment.center, children: Template())`
Stacks children on top of each other (later children on top). Layout: mirrors max width/height of highest-priority children.
```kscript
import { ZStack, Rectangle, Frame } from ui
export var main = ZStack {
    Rectangle(color: Color(0xFF8E0000)) with { Frame(width: 128, height: 128) }
    Rectangle(color: Color(0xFFFF5F52)) with { Frame(width: 64, height: 64) }
}
```

## Modifiers

Modifiers are applied via `with { ... }` blocks on components.

### Background
`Background(alignment: Alignment = Alignment.center, children: Template())`
Layers the given components behind this component; mirrors its size and suggests it to background children. Multiple children align as a group.
```kscript
import { Background, Text, Padding, Rectangle } from ui
export var main = Text("Hello, world!") with {
    Padding(8)
    Background { Rectangle(color: Color(0xFF123456)) }
}
```

### Clipped
`Clipped(_ clip: Bool = true)`
Clips the component to its bounding frame.
```kscript
import { Clipped, Image, Frame } from ui
export var main = Image("my_image.png") with {
    Frame(width: 50, height: 50)
    Clipped()
}
```

### Drag
`Drag(start: (DragEvent) -> (DragInfo), complete: (DropAction) -> (), minimum_distance: Float = 10, children: Template() = template() {})`
Enables drag-and-drop data transfer between/within applications. `start` returns a DragInfo with the data; `complete` (optional) receives the performed DropAction (`DropAction.cancel` if cancelled); `minimum_distance` = cursor movement before gesture responds (important when combining with TapGesture); `children` = drag cursor visual (OS/data-based if omitted).
```kscript
import { Rectangle, Drag, DragInfo, DropAction } from ui
export var main = Rectangle(color: Color(0xFFFFFF00)) with {
    Drag(start: fun (event) {
        return DragInfo(
            data: ["text/plain": "Ut queant laxis"],
            supported_actions: [DropAction.copy, DropAction.move],
        )
    })
}
```

`class DragEvent` properties: `position: Point` (pointer position in component coordinates at drag start), `frame: Rect`, `modifiers: KeyboardModifiers`.

`DragInfo(data: [String: String], supported_actions: [DropAction] = [DropAction.copy])` — `data` maps MIME types to data strings (multiple entries allowed for multiple formats).

### DragGesture
Pointer down/move/up sequence, used for sliders and knobs (not for Drag & Drop). Two constructors:
- `DragGesture(_ handler: (DragGestureEvent) -> ())` — called on pointer move.
- `DragGesture(minimum_distance: Float = 10, start: (DragGestureEvent) -> (), update: (DragGestureEvent) -> (), complete: (DragGestureEvent) -> ())` — `update` is also called with `start`, so both are only needed for start-specific work.
```kscript
import { DragGesture, DragGestureEvent, Rectangle, Point, Frame, Position } from ui
component DraggableBox {
    position: Point = Point(x: 0, y: 0)
    Rectangle(color: Color(0xFFFF0000)) with {
        Frame(width: 50, height: 50)
        Position(x: self.position.x, y: self.position.y)
        DragGesture(minimum_distance: 0, update: fun (event: DragGestureEvent) {
            self.position = Point(x: event.position.x - 25, y: event.position.y - 25)
        })
    }
}
export var main: Component = DraggableBox()
```

`class DragGestureEvent` properties: `position: Point`, `start_position: Point`, `frame: Rect`, `delta: Point` (delta to last pointer event), `modifiers: KeyboardModifiers`.

### Drop
Enables receiving drag-and-drop. Two constructors:
- `Drop(can_accept: (DropEvent) -> (Bool), enter: (DropEvent) -> (), update: (DropEvent) -> (DropAction?), exit: () -> (), perform: (DropEvent, DropAction) -> (Bool))`
- `Drop(of: [String] = [], enter: (DropEvent) -> (), update: (DropEvent) -> (DropAction?), exit: () -> (), perform: (DropEvent, DropAction) -> (Bool))` — `of` is a list of accepted MIME types.

`enter`/`update`/`exit` are optional; `update` returning `nil` keeps the proposed action; `perform` is called on a valid drop.
```kscript
import { Rectangle, Drop } from ui
export component Main {
    text: String = "Waiting..."
    Rectangle(color: Color(0xFF80C0D0)) with {
        Drop(of: ["text/plain"], perform: fun (event, action) {
            self.text = "\{action}: \{event.data(format: "text/plain")}"
            return true
        })
    }
}
export var main = Main()
```

`class DropEvent` — methods: `has_format(_ mime_type: String) -> (Bool)`, `data(format: String) -> (String)` (throws internal exception if format unavailable), `supports_action(_ action: DropAction) -> (Bool)`; properties: `position: Point`, `frame: Rect`.

```kscript
enum DropAction { copy, move, link, cancel }
```

### FontFamily
`FontFamily(_ family: FontFamilyName?)`
Overrides the default font family inherited by descendent text components; `nil` inherits from above.
```kscript
import { FontFamily, Text, VStack, load_font } from ui
var custom_font = load_font(["Nunito-Regular.ttf"])
export var main = VStack {
    Text("Default")
    VStack { Text("Overriden") } with { FontFamily(custom_font) }
}
```

### FontSize
`FontSize(_ size: Int?)`
Overrides the default font size inherited by descendent text components; `nil` inherits.
```kscript
import { FontSize, Text, VStack } from ui
export var main = VStack { Text("Big") } with { FontSize(24) }
```

### Frame
Invisible frame applying layout constraints. Two constructors:
- `Frame(width: Float? = nil, height: Float? = nil, alignment: Alignment = Alignment.center)` — fixed frame (min == max); unset dimensions mirror the child.
- `Frame(min_width: Float? = nil, max_width: Float? = nil, min_height: Float? = nil, max_height: Float? = nil, alignment: Alignment = Alignment.center)` — lower/upper limits.
```kscript
import { Frame, Rectangle } from ui
export var main = Rectangle(color: Color(0xFF000000)) with { Frame(width: 100, height: 100) }
```

### Hidden
`Hidden(_ hidden: Bool = true)`
Hides the component (inverse of Visible); hidden components still occupy space, don't render and can't be interacted with.
```kscript
import { Hidden, Text } from ui
export var main = Text("Secret") with { Hidden() }
```

### Hover
Track pointer enter/leave/move. Two constructors:
- `Hover(_ hovered: $<Bool>)` — binding for hover state.
- `Hover(enter: (HoverEvent) -> (), move: (HoverEvent) -> (), exit: (HoverEvent) -> ())`
```kscript
import { Hover, Text } from ui
component HoverText {
    hovered: Bool = false
    Text(self.hovered ? "Hovered" : "Not Hovered") with { Hover(self.$hovered) }
}
export var main: Component = HoverText()
```

`HoverEvent` properties: `position: Point`, `frame: Rect`, `modifiers: KeyboardModifiers`.

### LetterSpacing
`LetterSpacing(_ spacing: Float?)`
Overrides letter spacing (px, can be negative) inherited by descendent text; `nil` inherits. Introduced in kscript 1.8.
```kscript
import { LetterSpacing, Text } from ui
export var main = Text("Wide") with { LetterSpacing(3.0) }
```

### LineHeight
`LineHeight(_ factor: Float?)`
Overrides line height multiplier for descendent Text (e.g. 1.5 = 150% of natural line height); `nil` inherits. Introduced in kscript 1.8.
```kscript
import { LineHeight, Text } from ui
export var main = Text("Two\nlines") with { LineHeight(1.5) }
```

### LineLimit
`LineLimit(_ max_line_count: Int?)`
Overrides max line count for descendent text; `nil` inherits.
```kscript
import { LineLimit, Text } from ui
export var main = Text("Line 1\nLine 2") with { LineLimit(1) }
```

### MultilineTextAlignment
`MultilineTextAlignment(_ alignment: HorizontalAlignment?)`
Overrides multiline text alignment for descendent text; `nil` inherits.
```kscript
import { MultilineTextAlignment, HorizontalAlignment, Text } from ui
export var main = Text("Line 1\nLonger Line 2") with {
    MultilineTextAlignment(HorizontalAlignment.left)
}
```

### Offset
Offsets a component from its determined position (mirrors its size). Constructors: `Offset(x: Float)`, `Offset(y: Float)`, `Offset(x: Float, y: Float)`.
```kscript
import { Offset, Text } from ui
export var main = Text("Hello") with { Offset(x: 100, y: 50) }
```

### Opacity
`Opacity(_ opacity: Float)`
Applies opacity 0.0 (transparent) to 1.0 (opaque) to the component and children; applied to each descendent individually, so overlapping parts become visible.
```kscript
import { Opacity, Text } from ui
export var main = Text("Faded") with { Opacity(0.5) }
```

### Overlay
`Overlay(alignment: Alignment = Alignment.center, children: Template())`
Layers the given components in front of this component; mirrors its size and suggests it to children. Multiple children align as a group.
```kscript
import { Overlay, Text, Rectangle } from ui
export var main = Rectangle(color: Color(0xFF123456)) with {
    Overlay { Text("Hello, world!") }
}
```

### Padding
Adds empty space around the component; negative values invert the behavior. Constructors:
- `Padding(_ padding: Float)` — all sides.
- `Padding(horizontal: Float = 0, vertical: Float = 0)`
- `Padding(left: Float = 0, right: Float = 0, top: Float = 0, bottom: Float = 0)`
```kscript
import { Padding, Text } from ui
export var main = Text("Hello") with { Padding(8) }
```

### Popover
Displays content in a layer above all other components, anchored around the component. Two constructors (same params, differing `visible` type):
```kscript
Popover(
    visible: $<Bool>,   // auto-closes on outside click, sets binding to false
    direction: PopoverDirection = PopoverDirection.down,
    alignment: PopoverAlignment = PopoverAlignment.center,
    alignment_offset: Float = 0,   // shift from alignment line
    spacing: Float = 0,            // gap between anchor and popover
    children: Template(),
)
```
```kscript
// Introduced with kscript 1.3
Popover(visible: Bool, ...)  // never auto-closes; mouse events pass through background
```
If the popover doesn't fit in `direction` it is mirrored, then repositioned to fit the app frame; leading/trailing alignment flips if it doesn't fit. Popover content has the size of the entire scene available; no layout effect on the anchor.
```kscript
import { Popover, Text, TapGesture, TapGestureEvent } from ui
component ButtonWithToolTip {
    show_tip: Bool = false
    Text("Click Me") with {
        TapGesture(fun (event: TapGestureEvent) { self.show_tip = not self.show_tip })
        Popover(visible: self.$show_tip) { Text("You found me") }
    }
}
export var main: Component = ButtonWithToolTip()
```

```kscript
enum PopoverAlignment { leading, center, trailing }
enum PopoverDirection { up, down, left, right }
```

### Position
Absolutely positions the component from the top-left corner of the available space; fills the available area on the specified axis. Constructors: `Position(x: Float)`, `Position(y: Float)`, `Position(x: Float, y: Float)`.
```kscript
import { Position, Text } from ui
export var main = Text("Hello") with { Position(x: 100, y: 50) }
```

### Rotation
`Rotation(_ angle: Angle)`
Visually rotates the component (positive = clockwise); rendering only, does not affect layout bounding box.
```kscript
import { Rotation, Text } from ui
export var main = Text("Tilted") with { Rotation(Angle(degrees: 45)) }
```

### TapGesture
Pointer down & up within a small radius (buttons, checkboxes). Constructors:
- `TapGesture(_ single: (TapGestureEvent) -> ())`
- `TapGesture(down: (TapGestureEvent) -> (), up: (TapGestureEvent) -> (), single: (TapGestureEvent) -> (), double: (TapGestureEvent) -> (), cancel: () -> ())`

If both `single` and `double` are provided, `single` may fire on a timeout (the system double-click speed) to disambiguate; without `double`, every pointer up triggers `single`. `cancel` fires when another gesture takes over.
```kscript
import { TapGesture, TapGestureEvent, Rectangle, Frame } from ui
component TappableBox {
    color: Color = Color(0xFF336699)
    Rectangle(color: self.color) with {
        Frame(width: 50, height: 50)
        TapGesture(fun (event: TapGestureEvent) { self.color = Color(0xFF993366) })
    }
}
export var main: Component = TappableBox()
```

`class TapGestureEvent` properties: `position: Point`, `frame: Rect`, `modifiers: KeyboardModifiers`.

### TextColor
`TextColor(_ color: Color?)`
Overrides the default text color inherited by descendent text; `nil` inherits.
```kscript
import { TextColor, Text, VStack } from ui
export var main = VStack { Text("Tinted") } with { TextColor(Color(0xFFAA7799)) }
```

### Visible
`Visible(_ visible: Bool)`
Shows/hides the component (inverse of Hidden); hidden components still occupy space.
```kscript
import { Visible, Text } from ui
export var main = Text("Second") with { Visible(false) }
```

## Classes

### FontFamilyName
`class FontFamilyName` — a font family loaded with `load_font`.

### KeyboardModifiers
`class KeyboardModifiers` — properties: `alt: Bool` (option on macOS), `control: Bool` (command on macOS), `shift: Bool`.

### Path
`class Path` — vector path. Constructor: `Path()` (empty). Methods:
- `move(to position: Point)` — begins a new subpath
- `add_line(to position: Point)`
- `add_quad_curve(to position: Point, control: Point)`
- `add_cubic_curve(to position: Point, control1: Point, control2: Point)`
- `close_subpath()`

### Point
`class Point` — 2D point. Constructor: `Point(x: Float, y: Float)`. Properties: `x: Float` (get/set), `y: Float` (get/set). Method: `to_string() -> (String)`.

### Rect
`class Rect` — rectangular region. Constructor: `Rect(x: Float, y: Float, width: Float, height: Float)`. Properties: `x`, `y`, `width`, `height` (all Float, get/set). Method: `to_string() -> (String)`.

## Enums

```kscript
enum Alignment {
    top_left,
    top_center,
    top_right,
    center_left,
    center,
    center_right,
    bottom_left,
    bottom_center,
    bottom_right,
}

enum HorizontalAlignment { left, center, right }

enum VerticalAlignment { top, center, bottom }
```

Also defined with their features above: `DropAction` (copy, move, link, cancel), `PopoverAlignment` (leading, center, trailing), `PopoverDirection` (up, down, left, right), Canvas `CapStyle` (flat, round, square) and `JoinStyle` (round, bevel, miter).

## Constants

### font_weights
Pre-defined font weight values (all `Int`, get-only):
- `thin` = 100
- `extra_light` = 200
- `light` = 300
- `normal` = 400
- `medium` = 500
- `demi_bold` = 600
- `bold` = 700
- `extra_bold` = 800
- `black` = 900

```kscript
import { Text, font_weights } from ui
export var main = Text("Hello, world!", font_weight: font_weights.bold)
```

## Functions

### load_font
`fun load_font(_ paths: [String]) -> (FontFamily)`
Loads font files (paths relative to the main file); files should be of the same family.
```kscript
import { Text, load_font } from ui
var my_font = load_font([
    "assets/Roboto-Regular.ttf",
    "assets/Roboto-Bold.ttf",
    "assets/Roboto-Italic.ttf",
])
export var main = Text("Hello, world!", font_family: my_font)
```
