<!-- Condensed from Komplete UI docs (Language Tour, Language Reference, Fundamentals) -->

# Komplete Script (kscript) Language Reference

Komplete Script is the language for building Kontakt instrument UIs and their business logic. It is **type-safe and statically typed**. Business logic is written imperatively; the UI is **declarative and reactive** — you describe what the UI looks like based on state, and the runtime keeps it in sync automatically. It works alongside KSP (Kontakt Script Processor), which handles real-time MIDI/audio; Komplete Script sits on top.

Files use the `.kscript` extension. Each file is a **module**. The entry point of an instrument is an exported `main` variable of type `Component`:

```kscript
import { Text } from ui

export var main: Component = Text("Hello, world!")
```

`print(...)` is always available without import (output goes to Creator Tools).

## Comments

- Line comments: `//` to end of line.
- Block comments: `/*` ... `*/` — **may be nested**.

```kscript
var x = 5 // line comment
/*
    Block comment.
    /* Nested block comment is legal. */
*/
```

## Naming Rules

Case is **enforced by the compiler**, not a style convention:

- **Symbols** — variables, properties, state, functions, methods, parameters, enum cases, module names — must **start with a lowercase letter**.
- **Types** — classes, components, modifiers, enums, type aliases — must **start with an uppercase letter**.

```kscript
var title = ""          // ok
var TITLE = ""          // error — symbol starts uppercase
class Vec2 { }          // ok
class vec2 { }          // error — type starts lowercase
```

Convention within those rules: `snake_case` for symbols, `PascalCase` for types.

## Variables

Declare with `var`. A variable **must be initialized at declaration** and can be reassigned (same type only). There is no `let`/`const`.

```kscript
var count = 0            // type inferred as Int
var explicit_float: Float = 42
count = 1                // reassignment ok
```

Type annotation syntax: `var name: Type = value`. Annotations are required when the initializer carries insufficient type information:

```kscript
var optional_int: Int? = nil
var empty_array: [Int] = []
var empty_map: [String: Int] = [:]
```

**Type conversions are strict.** The ONLY implicit conversion is `Int` → `Float`:

```kscript
var i = 42
var f: Float = i   // ok — Int to Float
i = f              // error — Float does not convert to Int
var s: String = i  // error — no implicit conversion to String
var s2 = "\{i}"    // string interpolation is the way to stringify
```

**`var` variables are NOT reactive.** Reassigning one triggers no UI updates. Reactivity only applies to class properties and component/modifier state (see Reactivity).

## Types

Primitive types:

| Type     | Description                    | Literals                    |
|----------|--------------------------------|-----------------------------|
| `Bool`   | Boolean                        | `true`, `false`             |
| `Int`    | 64-bit integer                 | `42`, `-7`, `0xFF0000`      |
| `Float`  | 64-bit floating point          | `3.14`, `1.5e-3`, `2.0E6`   |
| `String` | Unicode character sequence     | `"Hello"`, `""`             |

Collection types:

- Array: `[ElementType]` — ordered, growable, zero-indexed. Grows via `append`.
- Map: `[KeyType: ValueType]` — unordered key-value pairs. Lookup always returns an optional (`ValueType?`). Assigning `nil` to a key **removes** the entry.

```kscript
var scores: [Int] = [10, 20, 30]
scores[1] = 99
scores.append(40)

var lookup: [String: Int] = ["a": 1, "b": 2]
var value = lookup["a"]  // type: Int?
lookup["c"] = 3
lookup["a"] = nil        // removes "a"
```

Optional types: suffix any type with `?` (e.g. `String?`). Absence is `nil`.

Function types: `(ParamTypes) -> (ReturnTypes)`:

```kscript
var predicate: (Int) -> (Bool) = fun (arg) { return arg > 0 }
var callback: () -> () = fun () {}
var splitter: (String) -> (String, String) = fun (arg) { return arg, arg }
```

Type aliases with the `type` keyword (exportable):

```kscript
type Score = Int
export type PlayerID = Int
type Lookup = [String: Int]
```

## Literals

- **Integers**: decimal `42`, `0`; hexadecimal with `0x`/`0X` prefix: `0xFF0000`, `0xDEADBEEF`.
- **Floats**: require at least one digit on each side of the decimal point: `3.14`, `0.0`. Exponent notation with `e`/`E` and optional sign: `1.5e10`, `1.5e+10`, `1.5e-3`, `2.0E6`.
- **Float constants**: `nan` (Not a Number), `infinity` (positive infinity).

```kscript
var a: Float = infinity
var b: Float = nan
```

- **Strings**: double quotes: `"Hello"`. Interpolation: `"\{expression}"` (backslash before opening brace).
- **Booleans**: `true`, `false`.
- **Arrays**: `[1, 2, 3]` — trailing comma allowed. Empty: `[]` (needs a type annotation).
- **Maps**: `["Alice": 30, "Bob": 25]` — trailing comma allowed. Empty map: `[:]` (needs annotation; distinguishes it from empty array).
- **Nil**: `nil` — assignable to any optional type.

## Operators

Arithmetic: `+`, `-` (binary and unary negation), `*`, `/`, `^` (**power/exponentiation**), `%` (modulo).

`Int / Int` truncates; use a `Float` operand for a floating result:

```kscript
var a = 7 / 2       // 3  (truncated)
var b = 7.0 / 2     // 3.5
```

Comparison (result `Bool`): `==`, `!=`, `<`, `>`, `<=`, `>=`.

Identity: `===` — same **class instance** (reference identity), class instances only. Two distinct instances with equal values are not identical.

Logical (Bool operands, **keyword operators**): `not` (unary), `and`, `or`. There is no `!`, `&&`, or `||` for logic. `and`/`or` short-circuit.

Bitwise (Int): `&` AND, `|` OR, `~` **XOR when binary**, `~a` NOT/complement when unary, `<<` left shift, `>>` right shift. (Note: `~` is XOR, not `^` — `^` is power.)

```kscript
var flags = 10  // 0b1010
var mask  = 12  // 0b1100
print("\{flags & mask}")  // 8
print("\{flags ~ mask}")  // 6  (XOR)
print("\{~flags}")        // -11 (complement)
```

Ternary conditional: `condition ? value_if_true : value_if_false`. Condition must be `Bool`; both branches must produce a compatible type; result may be optional:

```kscript
var label = score > 100 ? "Won" : "Lost"
var opt: String? = condition ? "value" : nil
```

Assignment: `=`. It does not produce a value (no chained/expression assignment). There are no compound assignment operators documented (`+=` etc.) — write `x = x + 1`.

### Chaining onto a constructor call

A constructor call cannot be followed directly by `.` — wrap it in parentheses first (or bind it to a variable):

```kscript
Color(0xFFFFFFFF).opacity(0.5)      // error — no direct chaining onto a constructor
(Color(0xFFFFFFFF)).opacity(0.5)    // ok — extra parentheses
var white = Color(0xFFFFFFFF)       // ok — bind, then chain
var faded = white.opacity(0.5)
```

This applies to property access too (`(Vec2(x: 1.0, y: 2.0)).x`), and to every type — `Color`, `Angle`, `Range`, your own classes. Ordinary function and method calls chain fine (`make_color().opacity(0.5)`) — only constructor calls need the parentheses.

## Control Flow (imperative)

Conditions must be `Bool` expressions — `if score { }` is a **compile error**; there is no implicit comparison against 0/nil.

```kscript
if score >= 90 {
    print("A")
} elseif score >= 75 {   // note: single keyword "elseif", not "else if"
    print("B")
} else {
    print("C")
}
```

Loops — `for-in` over collections or `Range`, and `while`:

```kscript
for fruit in ["Apple", "Banana"] {
    print(fruit)
}

for i in Range(5) {      // 0, 1, 2, 3, 4
    print("\{i}")
}

var i = 0
while i < 3 {
    print("\{i}")
    i = i + 1
}
```

`break` exits `for-in` or `while` loops immediately. (No `continue` is documented.)

`do` creates an explicit scope block:

```kscript
var x = 1
do {
    var x = 2
    print("\{x}") // 2
}
print("\{x}")     // 1
```

## Optionals

`T?` holds either a `T` value or `nil`.

```kscript
var name: String? = "Alice"
name = nil
```

Check with `!= nil`, then **force-unwrap** with a postfix `!` on any expression yielding an optional:

```kscript
if name != nil {
    print("\{name!}")
}
print("\{find("Alice")!}")   // unwrap directly on a call result
```

Force-unwrapping `nil` is a **runtime error** — always check first. Map lookups return optionals (`map[key]` has type `Value?`). There is no documented `if let`, optional chaining (`?.`), or nil-coalescing operator — the only tools are `!= nil` checks, `!`, and ternaries producing optionals.

## Functions

Declare with `fun`. Return type is written `-> (Type)` after the parameter list — the parentheses around the return type are part of the syntax.

```kscript
fun double(_ x: Int) -> (Int) {
    return x * 2
}
```

### Argument Labels

By default the parameter name is the argument label at the call site (`greet(name: "Alice")`). A custom label goes **before** the parameter name; `_` makes the parameter positional (no label):

```kscript
fun clip(_ value: Int, from min: Int, to max: Int) -> (Int) {
    if value < min { return min }
    if value > max { return max }
    return value
}
var clamped = clip(42, from: 0, to: 10)
```

- Positional (`_`) parameters must appear in declaration order at the call site.
- Named parameters may be given **in any order**: `clip(42, to: 10, from: 0)` is valid.

### Default Parameter Values

```kscript
fun greet(name: String, greeting: String = "Hello") {
    print("\{greeting}, \{name}!")
}
greet(name: "Alice")   // "Hello, Alice!"
```

A defaulted parameter may appear anywhere in the parameter list.

### Return

- No return value: use `-> ()` or omit the return type entirely.
- Multiple return values: list types in the return type; separate values with commas after `return`; destructure with comma-separated `var` names:

```kscript
fun min_max(_ values: [Int]) -> (Int, Int) {
    return values[0], values[1]
}
var low, high = min_max([3, 1])
```

### Nested Functions, Function Values, Closures

Functions can be nested (with access to the enclosing scope), stored in variables, passed as arguments, and returned:

```kscript
fun make_adder(by amount: Int) -> ((Int) -> (Int)) {
    fun adder(_ x: Int) -> (Int) {
        return x + amount
    }
    return adder
}
var add_ten = make_adder(by: 10)
```

Closures are anonymous `fun` expressions and capture the surrounding scope. When the target type is known, parameter and return types can be omitted:

```kscript
var result = apply(5, transform: fun (x) { return x * 2 })
var double: (Int) -> (Int) = fun (x) { return x * 2 }
```

Coercing a named function into a closure value **loses its argument labels and default values** — all arguments must then be passed positionally:

```kscript
var fn = greet
fn("Alice", "Hi")    // ok — positional only
fn(name: "Alice")    // error — labels not available
fn("Alice")          // error — defaults not available
```

## Classes

Declare with `class`. Classes combine stored/computed properties and methods. **Methods are declared without the `fun` keyword.** `self` is implicit and refers to the instance.

```kscript
class Counter {
    count: Int = 0                 // stored property with default

    increment() {                  // method — no `fun`
        self.count = self.count + 1
    }

    value() -> (Int) {
        return self.count
    }
}
```

### Member Order (enforced)

Members must appear in this order — it is a language rule, not a style preference:

1. properties (stored and computed)
2. constructors
3. methods

A method declared before a property is a compile error.

### Properties

- Stored property: `name: Type` or `name: Type = default`. Properties without defaults must be supplied at construction.
- Read-only computed property — body directly after the type:

```kscript
class Circle {
    radius: Float = 1.0
    circumference: Float {
        return 2.0 * 3.14159 * self.radius
    }
}
```

- Read-write computed property — explicit `get`/`set` blocks; `set(paramName)` names the incoming value:

```kscript
class Slider {
    raw_value: Float = 0.0
    percentage: Float {
        get { return self.raw_value * 100.0 }
        set(p) { self.raw_value = p / 100.0 }
    }
}
```

Computed properties are part of the reactivity system: re-evaluated when dependencies change, not on every access.

### Constructors

Every class gets a generated **default constructor** taking all stored properties as named arguments (defaulted ones optional):

```kscript
var v = Vec2(x: 0.5, y: 1.0)
```

Custom constructors use the `constructor` keyword and **must delegate to the default constructor via `: (…)`**:

```kscript
class BoundingBox {
    top_left: Vec2
    bottom_right: Vec2

    constructor(x: Float, y: Float, width: Float, height: Float) : (
        top_left: Vec2(x: x, y: y),
        bottom_right: Vec2(x: x + width, y: y + height),
    )
}
```

- Defining any custom constructor **hides** the generated default constructor.
- A custom constructor may add a body block `{ … }` after the delegation for setup steps.
- Multiple constructors are allowed if their parameter signatures differ.
- Constructor parameters follow function parameter rules (labels, `_`, defaults).

### Method Overloading (Kontakt 8.12)

A class can define multiple methods with the same name, as long as their parameters differ — in number, in types, or in argument labels. The compiler selects the matching overload based on the arguments at the call site:

```kscript
class Playlist {
    titles: [String] = []

    add(_ title: String) {
        self.titles.append(title)
    }

    add(_ titles: [String]) {
        self.titles.append(titles)
    }
}

var playlist = Playlist()
playlist.add("Intro")             // calls add(_ title: String)
playlist.add(["Verse", "Chorus"]) // calls add(_ titles: [String])
```

Methods that differ only in their return type are not valid overloads. Overloading is only supported for methods — free functions cannot be overloaded.

### String Interpolation of Instances

Interpolating an instance prints its stored properties: `Vec2(x: 3.0, y: 4.0)` (computed properties excluded). Define `to_string() -> (String)` to customize.

### Subscript Operator (Kontakt 8.11)

Define a method named `at` with exactly one parameter to enable `obj[expr]` reads; define `assign` with exactly two parameters (value first, then subscript) and no return value to enable `obj[expr] = value`. Both are also callable directly as normal methods.

```kscript
class NumberList {
    values: [Int] = []

    at(_ index: Int) -> (Int) {
        return self.values[index]
    }

    assign(_ value: Int, at index: Int) {
        self.values[index] = value
    }
}
var list = NumberList(values: [10, 20, 30])
print("\{list[1]}")  // 20
list[1] = 99
```

### Reference Semantics

Classes are **reference types**: assignment and argument-passing share the same instance; mutation through one reference is visible through all. `===` tests identity. This makes a globally declared class instance a good shared reactive data model.

```kscript
var a = Counter()
var b = a
b.count = 10
print("\{a.count}") // 10
```

## Enumerations

```kscript
enum Direction {
    north,
    south,
    east,
    west,
}

var heading = Direction.north            // cases prefixed with the type name
if heading == Direction.north { }        // == and != supported
print("\{heading}")                      // interpolation prints the case name: "north"
```

Enum cases use lowercase names. No associated values or raw values are documented.

## Components

Components are the UI building blocks. Declare with `component Name { … }`. The body contains **component expressions** (the UI tree) plus declarations of properties/bindings/state/methods. A component body cannot be empty. Refer to members through `self` (implicit availability inside).

```kscript
import { Text } from ui

component Welcome {
    @property text: String

    Text(self.text)
}

export var main: Component = Welcome(text: "Hello!")
```

### Member Order (enforced)

Components and modifiers require this order:

1. `@property` / `@binding` declarations (may interleave with each other)
2. constructors
3. state and methods (may interleave with each other)
4. **child component expressions — always last**

The UI tree goes at the bottom of the body; a `@property` or method declared after a component expression is a compile error.

```kscript
component Card {
    @property title: String     // 1. properties/bindings
    @binding open: Bool

    constructor(title: String, open: $<Bool>) : (   // 2. constructors
        title: title,
        open: open,
    )

    hovered: Bool = false       // 3. state / methods
    toggle() {
        self.open = not self.open
    }

    VStack {                    // 4. children — last
        Text(self.title)
    }
}
```

### @property (parent → child, read-only)

Declared as `@property name: Type`. Passed as named arguments at the call site. **Read-only inside the component** — assigning to a property is an error. Properties are also inaccessible from outside (`welcome.text` does not work). A property passed to a component is an expression; if it references reactive state it updates automatically.

Callback properties (child → parent information flow) are properties of function type:

```kscript
component Button {
    @property label: String
    @property on_press: (Bool) -> ()

    Text(self.label) with {
        TapGesture(fun (event: TapGestureEvent) {
            self.on_press(event.modifiers.shift)
        })
    }
}
```

### State (internal, mutable, reactive)

State is declared like a class stored property — name, type, **required initial value** — with no annotation keyword. It is private to the component, cannot be set from outside, and is reactive: changing it re-evaluates every dependent expression.

```kscript
component Counter {
    count: Int = 0

    Text("\{self.count}") with {
        TapGesture(fun (_: TapGestureEvent) {
            self.count = self.count + 1
        })
    }
}
```

Computed state mirrors computed class properties — read-only shorthand or `get`/`set`:

```kscript
component Toggle {
    on: Bool = false

    label: String {
        return self.on ? "On" : "Off"
    }

    percentage: Float {
        get { return self.on ? 100.0 : 0.0 }
        set(p) { self.on = p > 50.0 }
    }

    Text(self.label)
}
```

### @binding (read-write reference)

`@binding name: Type` declares that the component receives a read-write reference to state owned elsewhere. Inside, it reads and writes like normal state — writes propagate back to the owner.

Pass a binding by prefixing the state/property name with `$`:

```kscript
component Checkbox {
    @binding checked: Bool

    Rectangle(color: self.checked ? Color(0xFFFFFFFF) : Color(0x66FFFFFF)) with {
        TapGesture(fun (_) {
            self.checked = not self.checked
        })
    }
}

component Main {
    checked: Bool = false

    Checkbox(checked: self.$checked)   // pass binding to own state
}
```

- `$` works on stored state, **read-write** computed state, and class properties (`model.$active`).
- A component forwards a received binding onward the same way: `Checkbox(checked: self.$checked)`.
- The binding type is written `$<T>` (e.g. `$<Bool>`) — needed in custom constructor signatures:

```kscript
component Checkbox {
    @binding checked: Bool

    constructor(label: String, checked: $<Bool>) : (
        checked: checked,
    )
    // ...
}
```

- Bindings are opaque: they cannot be created, read, or written directly in code — only obtained via `$` and consumed via `@binding`.

### Component Constructors

Like classes: a generated default constructor accepts all `@property` and `@binding` declarations as named arguments. Custom constructors use `constructor(...) : (…)` delegation, may be overloaded by signature, and follow function parameter rules. **Unlike class constructors, component constructors cannot have a body block.**

```kscript
component Card {
    @property title: String
    @property width: Float
    @property height: Float

    constructor(title: String, size: Float) : (
        title: title,
        width: size,
        height: size,
    )

    Text(self.title)
}
```

### Children and Trailing Blocks

A `@property` named exactly `children` with type `Template()` enables the trailing-block call syntax:

```kscript
component Row {
    @property children: Template()

    HStack(spacing: 8) {
        self.children()      // invoke the template to place the children
    }
}

export var main: Component = Row {
    Text("First")
    Text("Second")
}
```

Trailing-block parentheses rules — all equivalent when no other arguments:

```kscript
VStack(spacing: 8) { … }   // other args — parentheses required
VStack() { … }             // empty parens optional
VStack { … }               // parens omitted entirely
```

Any other template property (different name, or parameterized `Template(…)`) must be passed explicitly as a named argument using a `template` literal:

```kscript
export var main: Component = Layout(
    header: template () { Text("Title") },
    item: template (text) { Text(text) },
) {
    Text("First")
}
```

### Multiple Siblings, No Wrapper

A component body may produce multiple sibling components (directly, or via declarative `if`/`for`). All of them become **direct siblings in the parent container** — no intermediate wrapper node exists, so container properties like `spacing` apply to each individually.

### Exporting

`export component Name { … }` exports a component. The instrument entry point is `export var main: Component = SomeComponent()`.

## Templates

A template is a **deferred block of components**: its contents are created when invoked, not when defined. Type syntax lists parameter types:

```kscript
Template()             // no parameters
Template(String)       // one String parameter
Template(String, Int)  // two parameters
```

Template literal syntax: `template (params) { …component expressions… }`. Invoke a template property like a function: `self.item(text)`.

```kscript
component List {
    @property data: [String]
    @property item: Template(String)

    VStack {
        for text in self.data {
            self.item(text)
        }
    }
}

export var main: Component = List(
    data: ["Alpha", "Beta"],
    item: template (text) {
        Text(text)
    }
)
```

### No `var` inside templates or trailing blocks

A `template (…) { … }` literal and a trailing `{ … }` children block hold **component expressions only** — `var` declarations are not allowed in them. Compute the value outside and pass it in (as a template parameter, a component `@property`, or component state):

```kscript
// error — var inside a trailing children block
VStack {
    var label = "Track \{index}"
    Text(label)
}

// ok — compute outside, or inline the expression
Text("Track \{index}")

// ok — a template parameter carries the value in
List(
    data: names,
    item: template (name) {
        Text(name)
    },
)
```

Declarative `if`/`for` inside those blocks is fine — only variable declarations are rejected.

### Templates vs Functions (critical)

Calling a **function** in a reactive expression records dependencies on all its arguments — every argument change re-invokes it, and if it builds components, they are destroyed and recreated each time (loses local state, breaks animations, costs performance). **Templates lazily forward their parameters**: invoking one creates no dependency in the surrounding context; component instances are reused and only the changed parts update.

Rule: use `Template(…)` whenever the purpose is producing components; use function properties only for callbacks/logic.

```kscript
@property item: (String) -> (Component)  // BAD for UI — recreates components
@property item: Template(String)         // GOOD — reuses components
```

## Modifiers

A modifier transforms a component. Apply modifiers with a `with` block after a component expression:

```kscript
import { Text, Padding, Background, Rectangle } from ui

export var main: Component = Text("Hello") with {
    Padding(8)
    Background {
        Rectangle(color: Color(0xFF222222))
    }
}
```

**Order matters**: modifiers apply top to bottom; each later modifier wraps the result of all previous ones. The same modifier type may be applied multiple times (e.g. `Background`, then `Padding`, then another `Background`).

### `with` Applies to Any Component Expression

A `with` block is not limited to constructor calls — it follows **any expression producing a component**, and takes the same modifier list in every case:

```kscript
self.children() with { Padding(8) }   // template invocation — per child, one shared modifier instance
self.item(text) with { Padding(8) }   // parameterized template invocation
self.highlight with { Padding(8) }    // Component-typed property or state
self.slides[0] with { Padding(8) }    // element of a [Component]
labelled(text: "x") with { … }        // function/method call returning Component
(flag ? a : b) with { Padding(8) }    // parenthesized ternary
```

### Custom Modifiers

Declare with the `modifier` keyword. The modified component is available inside the body as the implicit variable **`child`**. Bodies may produce multiple siblings and use declarative `if`/`for`. Modifiers support `@property`, `@binding`, and state just like components.

```kscript
modifier ColoredBackground {
    @property color: Color

    child with {
        Background {
            Rectangle(color: self.color)
        }
    }
}

export var main: Component = Text("Hello") with {
    ColoredBackground(color: Color(0xFF222222))
}
```

Exportable: `export modifier Name { … }`.

### Modifiers on Multi-Child Components

When applied to a component that yields multiple children, the modifier is applied **separately to each child**, but it is the **same modifier instance** for all — modifier state is shared across every wrapped child (e.g. a Highlight modifier's `active` state toggles all children's backgrounds together).

## Declarative Control Flow

Component bodies, modifier bodies, and template bodies support **declarative** `if`/`elseif`/`else` and `for` — they produce components (each expression contributes to the output automatically) rather than executing side effects. NOT available in imperative code (function bodies, class methods).

```kscript
component Status {
    @property value: Int
    @property names: [String]

    if self.value > 100 {
        Text("High")
    } elseif self.value > 50 {
        Text("Medium")
    } else {
        Text("Low")
    }

    for name in self.names {
        Text(name)
    }
}
```

All produced components appear as **direct siblings** in the parent container — no wrapper is inserted. Conditions of declarative `if` automatically track their reactive dependencies (the branch re-renders when the condition's inputs change).

## Modules, Import & Export

A module = one `.kscript` file. Module names must **start with a lowercase letter** and may contain only lowercase letters, digits, and underscores.

### Export

Prefix any top-level declaration with `export`:

```kscript
export var version: String = "1.0"
export class Model { … }
export enum Page { main, settings }
export component Header { … }
export modifier Highlight { … }
export type PlayerID = Int
```

Re-export from another module:

```kscript
export { Rectangle, Text } from ui        // re-export named symbols
export * as util from util                // re-export whole module as a namespace symbol
```

### Import

```kscript
import { Text, HStack } from ui                    // named
import { Text as Label, HStack as Row } from ui    // rename with `as`
import * from my_module                            // wildcard
import * as ui from ui                             // namespace: use as ui.Text("...")
```

Importing from a nested namespace uses dot-separated names and **requires** a rename:

```kscript
import { controls.knob.Knob as Knob } from my_module
```

### Directory Structure

Modules may live in subdirectories; import paths are dot-separated and **always resolved from the project root** (the `komplete_scripts` directory), never relative to the importing file:

```kscript
import * from components.tabbar
import * from kontakt_components.base.button_base
```

### Folder Modules

A directory containing an `init.kscript` acts as a module: importing the folder path resolves to that file. `init.kscript` typically re-exports the folder's internals:

```kscript
// controls/init.kscript
export { Knob } from controls.knob
export { Fader } from controls.fader

// consumer
import { Knob, Fader } from controls
```

## Reactivity (how the UI updates)

Reactive storage: **class properties** and **component/modifier state** (including computed) are reactive; `@property`/`@binding` expressions and declarative bodies/templates track their dependencies. When any tracked value changes, every dependent expression re-evaluates and the UI updates automatically. Plain `var` variables are NOT reactive.

- Dependencies are tracked no matter how indirectly accessed — through methods, computed state, or class computed properties:

```kscript
component Example {
    state: Int = 4
    derived: Int {
        return self.state + 10
    }
    offset(by delta: Int) -> (Int) {
        return self.derived + delta
    }

    Text("Hello", size: self.offset(by: 5))  // updates when state changes
}
```

- A globally declared class instance works as a shared reactive data model:

```kscript
class Model {
    checked: Bool = false
}
var model = Model()

export component Checkbox {
    Rectangle(color: model.checked ? Color(0xFFFF0000) : Color(0xFF00FF00)) with {
        TapGesture(fun (event) {
            model.checked = not model.checked
        })
    }
}
```

- Using a value inside a component's property makes only **that property** reactive, not the creation of the component itself.
- **Every** value read inside a reactive expression is tracked — even one read only for a `print`. Reading an unrelated state inside a computed state makes it re-evaluate when that state changes.

### Reactivity Pitfalls

1. **`var` in components does not work** — a component reading a global `var` in an expression will never update when the `var` changes. Use component state or a class property instead.
2. **UI code must be side-effect free.** The order and count of re-evaluations is NOT guaranteed and may change between Kontakt versions. Never mutate anything (including globals) inside computed state/properties.
3. **Never create components from functions** (`(Args) -> (Component)` properties or method calls in the body) — the component is recreated whenever any argument changes. Use `Template(…)` instead.

## Layout Fundamentals

Layout rule: **Parent proposes size. Child chooses size. Parent positions child.** A parent (e.g. `Frame`, a stack) proposes an available size to each child; the child decides its own size (a `Rectangle` fills the proposal; `Text` takes only what it needs); the parent then positions the child (e.g. centered) and derives its own size from its children.

Container components from `ui`:

- `HStack(spacing: N) { … }` — children on a horizontal line.
- `VStack(spacing: N) { … }` — vertical line.
- `ZStack { … }` — children layered on top of each other (later children on top); considers all children equally to determine its own size; children that don't fit are not constrained by earlier ones.

Common layout modifiers:

- `Frame(width: N, height: N)` — proposes a fixed size.
- `Padding(N)` — pads all sides equally.
- `Overlay { … }` — layers components **in front of** the modified component; the modified component's size is unchanged and is proposed to the overlaid components (content that doesn't fit will wrap/clip).
- `Background { … }` — layers components **behind** the modified component; the foreground component's size drives the background's proposed size.

```kscript
import { Rectangle, Text, Frame, Overlay } from ui

export var main = Rectangle(color: Color(0xFFDFDFDF)) with {
    Frame(width: 100, height: 100)
    Overlay {
        Text("Hello Long Text")   // wraps to fit the 100x100 proposal
    }
}
```

Contrast with `ZStack`: in a `ZStack` the text is not constrained to the rectangle's size — all children are sized independently.

### Don't add Frame just to align

A `Frame` only earns its place when a child needs a size the layout wouldn't otherwise give it. If a sibling already fixes the container size — e.g. a full-area `Rectangle` in a `ZStack`, or the modified component in `Background`/`Overlay` — the parent already proposes that size and positions the remaining children by its `alignment`. Sizing those children with an explicit `Frame` for placement is redundant; set the container/stack `alignment` (or `Position`) instead.

```kscript
import { ZStack, Rectangle, Text, Alignment } from ui

// Redundant: Frame only used to push text to a corner
ZStack {
    Rectangle(color: bg)                                     // fills, sets ZStack size
    Text("v1.0") with { Frame(width: W, height: H) }         // Frame does nothing useful here
}

// Better: let the parent align it
ZStack(alignment: Alignment.bottom_right) {
    Rectangle(color: bg)
    Text("v1.0")
}
```

Reach for `Frame` only when the child's own size matters (fixed control size, spacer, min hit area).

### Expanding Frame — fill available space, don't hard-code

To make a child fill the space its parent proposes, use `max_width: infinity` / `max_height: infinity` (`Frame(min_width:, max_width:, min_height:, max_height:, alignment:)`) instead of hard-coding pixel sizes. `infinity` means "take as much as offered" — the child grows to the parent's proposal and stays responsive when the layout resizes. `alignment` positions the child within that expanded frame.

```kscript
import { HStack, Text, Rectangle, Background, Frame } from ui

// Equal-width cells splitting a row: each stretches to take half.
// A Spacer can't do this — it only claims leftover gaps, it can't size visible children.
HStack(spacing: 4) {
    Text("Left") with {
        Frame(max_width: infinity)
        Background { Rectangle(color: a) }
    }
    Text("Right") with {
        Frame(max_width: infinity)
        Background { Rectangle(color: b) }
    }
}

// Expand to fill both axes, pin the (self-sizing) content top-left
Text("Title") with {
    Frame(max_width: infinity, max_height: infinity, alignment: Alignment.top_left)
}
```

Combine with `min_*` for a floor and `max_*: infinity` for a stretchy ceiling. Prefer this over recomputing fixed sizes whenever the target is "as big as the parent allows".

## Gestures

Two gestures exist: `TapGesture` and `DragGesture` (from `ui`). Attach them as modifiers in a `with` block.

```kscript
Rectangle(color: Color(0xFF333333)) with {
    TapGesture(
        down: fun (event: TapGestureEvent) { print("Tap Down") },
        up: fun (event: TapGestureEvent) { print("Tap Up") },
        single: fun (event: TapGestureEvent) { print("Tap Single") },
        cancel: fun () { print("Tap Cancelled") }
    )
    DragGesture(
        start: fun (event: DragGestureEvent) { print("Drag Start") },
        update: fun (event: DragGestureEvent) { print("Drag Update") },
        complete: fun (event: DragGestureEvent) { print("Drag Complete") }
    )
}
```

A `TapGesture` may also take a single closure directly: `TapGesture(fun (event: TapGestureEvent) { … })`. Tap events expose `event.modifiers.shift` etc. Drag events expose `event.delta` (`.x`/`.y`) and `event.frame` (`.width`/`.height`).

`Hover(self.$hovered)` takes a `Bool` binding tracking hover state.

### Priority

Only one gesture can be **active** at a time; once active, it blocks all others until completion. Input goes first to the highest-priority gesture. Priority follows visual stacking:

- **Same component**: gestures are prioritized in order of definition — first listed has highest priority.
- **Ancestor vs descendant**: descendants (visually on top) have higher priority than ancestors.
- **Siblings**: visually higher (later) siblings take priority; a higher sibling's gesture **blocks** gestures on lower siblings entirely — combining gestures across siblings is not possible.

### Combining Tap and Drag

With a tap and a drag on the same component: a pointer-down triggers the tap's `down` handler, but neither gesture is active yet. Release near the start point → tap completes (`single`), drag never responds. Move beyond the drag threshold → tap `cancel` fires, then drag `start` + `update`, and `complete` on release.

`DragGesture(minimum_distance: 0, …)` normally starts on pointer-down, **except** when a higher-priority tap gesture exists — then the drag starts on first movement so the tap can still fire. If the zero-threshold drag is listed first (higher priority), the tap never triggers.

## Kontakt Integration Pattern

Connect to KSP controls via the `kontakt` package. KSP connections are fixed at load time, so declare them **globally**, not inside a component:

```kscript
import { VStack, Text, Arc, DragGesture, Padding } from ui
import { KSPKnob } from kontakt

var reverb_knob = KSPKnob(id: "reverb")   // connects to KSP ui_knob "reverb"

component ReverbSend {
    accumulated_delta: Float = 0.0        // carries sub-integer drag remainder

    VStack(spacing: 4) {
        Arc(
            color: Color(0xFF000000),
            start_angle: Angle(degrees: 135),
            angle: Angle(degrees: reverb_knob.normalized_value * 270),
        ) with {
            DragGesture(fun (event) {
                var dy = event.delta.y / event.frame.height
                self.accumulated_delta = self.accumulated_delta - dy * (reverb_knob.max - reverb_knob.min)
                var steps = self.accumulated_delta.to_int()
                if steps != 0 {
                    reverb_knob.value = (reverb_knob.value + steps).clamped(min: reverb_knob.min, max: reverb_knob.max)
                    self.accumulated_delta = self.accumulated_delta - steps
                }
            })
        }
        Text(reverb_knob.label)       // reactive: updates with KSP label
        Text("\{reverb_knob.value}")  // reactive: updates with KSP value
    } with {
        Padding(16)
    }
}

export var main: Component = ReverbSend()
```

`KSPKnob` exposes reactive `value` (Int), `min`, `max`, `normalized_value` (Float 0.0–1.0), and `label`. The `kontakt controls` package provides ready-made `Knob`/`Slider` components wrapping this pattern. Colors are constructed as `Color(0xAARRGGBB)` (e.g. `Color(0xFFFF0000)` opaque red).

## Gotchas (differences from mainstream languages)

1. **`elseif` is one keyword** — not `else if`, not `elif`.
2. **Logical operators are keywords**: `not`, `and`, `or`. `!` is only postfix force-unwrap; `&`/`|` are bitwise only.
3. **`^` is power (exponentiation), not XOR. `~` (binary) is XOR; `~` (unary) is complement.**
4. **Conditions must be `Bool`** — no truthiness, no implicit `!= 0` / `!= nil`.
5. **Only implicit conversion is `Int` → `Float`.** Everything else needs explicit conversion (e.g. `.to_int()`) or string interpolation `"\{x}"` for strings.
6. **String interpolation is `"\{expr}"`** — backslash + curly braces, not `${}` or `\( )`.
7. **Return types are parenthesized**: `-> (Int)`, `-> (Int, String)`; void is `-> ()` or omitted.
8. **Arguments are labeled by default** (Swift-style). Use `_` before the parameter name for positional. Named args may be reordered at the call site.
9. **Function values lose labels and defaults** — a function assigned to a variable must be called with all arguments, positionally.
10. **Class methods have no `fun` keyword**; top-level/nested functions require it.
11. **`self` is mandatory** for member access inside classes/components (`self.count`, not `count`).
12. **Custom constructors must delegate** with `constructor(...) : (props...)`, and defining one hides the generated default constructor. Component constructors cannot have a body.
13. **`var` is not reactive.** Component state, class properties, and bindings are. A component reading a plain `var` never updates.
14. **Component `@property` is read-only inside the component and inaccessible from outside.** Two-way access needs `@binding` + `$` prefix at the call site (`self.$state`, `model.$prop`).
15. **`Template(…)` vs function properties**: templates avoid dependency capture and component recreation. Never use `(Args) -> (Component)` properties for UI content.
16. **Trailing block only works for a `@property` named `children` of type `Template()`**; all other template properties are passed as `name: template (params) { … }` arguments.
17. **Multiple components in a body/`if`/`for` become direct siblings** in the parent container — no implicit wrapper.
18. **Modifier order matters** (top-to-bottom wrapping), and one modifier instance applied to a multi-child component shares its state across all children.
19. **Modifier bodies reference the wrapped component via the implicit `child` variable.**
20. **`with` works on any component expression**, not just constructor calls — `self.children() with { … }`, `self.item(text) with { … }`, `self.some_component with { … }`, `self.items[0] with { … }`.
21. **UI code must be side-effect free** — re-evaluation order/count is unspecified; any value read in a reactive expression becomes a tracked dependency (even reads inside `print`).
22. **Map lookups always return optionals; assigning `nil` deletes the key.** Force unwrap (`!`) on `nil` is a runtime crash.
23. **Empty collection literals need type annotations**; empty map is `[:]`, not `{}` or `[]`.
24. **Import paths resolve from the project root** (`komplete_scripts/`), never relative to the current file. Module filenames must be lowercase (letters, digits, underscores). Nested-namespace imports require `as` renames.
25. **Block comments nest.**
26. **`===` (identity) exists only for class instances**; `==` compares values.
27. **KSP connections (e.g. `KSPKnob`) must be declared globally** — they are fixed at load time and don't belong inside components.
28. **Higher (later) siblings block lower siblings' gestures entirely** — you cannot combine gestures across siblings.
29. Float literals need digits on both sides of the dot (`0.5`, not `.5`; `1.0`, not `1.`).
30. **Case is enforced**: symbols (variables, properties, functions, enum cases) must start lowercase; types (classes, components, modifiers, enums, aliases) must start uppercase. `var TITLE = ""` does not compile.
31. **No direct chaining onto a constructor call** — `Color(0xFFFFFFFF).opacity(0.5)` is illegal; write `(Color(0xFFFFFFFF)).opacity(0.5)` or bind to a variable first.
32. **Member order is enforced.** Classes: properties → constructors → methods. Components/modifiers: properties/bindings → constructors → state/methods → child components last.
33. **No `var` inside templates or trailing `{ … }` children blocks** — those take component expressions only. Pass values in via template parameters, properties, or state.
