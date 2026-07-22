<!-- Condensed from Komplete UI docs (Standard Library) -->

# kscript Standard Library Reference

## Print

```kscript
fun print(_ message: String) -> ()
```

Prints a debug message (can be read from MCP).

```kscript
print("Hello, KSP console!")
```

## Warning

```kscript
fun warning(_ message: String) -> ()
```

Prints a warning message (can be read from MCP).

```kscript
warning("Be careful")
```

## Error

```kscript
fun error(_ message: String) -> ()
```

Aborts the current call stack and prints an error message to Creator Tools. Code after `error()` in the aborted stack is not reached.

```kscript
if amount > balance {
    error("cannot withdraw more than \{balance}")
}
```

## Int

An integer value, e.g. `12`.

Methods:
- `clamped(min: Int, max: Int) -> (Int)` — restricts the value to `[min, max]`.
- `to_float() -> (Float)` — float representation.

```kscript
var b = 30.clamped(min: 0, max: 10) // 10
var f = 5.to_float() // 5.0
```

## Float

A floating point value, e.g. `3.14`.

Special values (literals): `infinity`, `nan` (invalid value, e.g. from `0/0`).

Properties:
- `is_finite: Bool` (get)
- `is_nan: Bool` (get)

Methods:
- `ceiled() -> (Float)` — least integer not less than value: `3.14 -> 4.0`, `-3.14 -> -3.0`.
- `clamped(min: Float, max: Float) -> (Float)`
- `floored() -> (Float)` — largest integer not greater than value: `3.14 -> 3.0`, `-3.14 -> -4.0`.
- `formatted(digits: Int) -> (String)` — fixed-point notation: `0.5.formatted(digits: 2)` -> `"0.50"`.
- `rounded() -> (Float)` — nearest integer, half-way cases rounded away from zero: `3.5 -> 4.0`, `-3.5 -> -4.0`.
- `to_int() -> (Int)` — integer part (truncation).

```kscript
var value = 3.1415
print("\{value.formatted(digits: 2)}") // Prints "3.14"
```

## String

A collection of unicode characters. All strings are expected to be UTF-8 encoded.

- Compare with `==` / `!=`.
- Escape sequences: `\n`, `\r` (since 8.9), `\\`, `\"`, `\u{030A}` (unicode codepoint, hex), `\{<expr>}` (interpolation).
- Combining codepoints (e.g. `\u{030A}`) merge with the leading character and do not count as separate characters for `length`.

Properties:
- `byte_count: Int` (get) — number of bytes stored.
- `is_empty: Bool` (get)
- `length: Int` (get) — number of unicode characters.

Methods:
- `appending(_ other: String) -> (String)`
- `contains(_ other: String) -> (Bool)`
- `has_prefix(_ prefix: String) -> (Bool)`
- `has_suffix(_ suffix: String) -> (Bool)`
- `index(of other: String, from: Int = 0) -> (Int?)` — first index of `other`, searching from the given character.
- `inserting(_ other: String, at index: Int) -> (String)`
- `lexicographically_precedes(_ other: String) -> (Bool)` — gotcha: only orders correctly for the latin alphabet.
- `lowercased() -> (String)`
- `prefix(_ max_length: Int) -> (String)`
- `removing_all(_ other: String) -> (String)`
- `removing_subrange(from: Int, length: Int? = nil) -> (String)` — `length` defaults to remainder after `from`.
- `replacing_all(_ target: String, with replacement: String) -> (String)`
- `reversed() -> (String)`
- `split(separator: String) -> ([String])`
- `substring(from: Int, length: Int? = nil) -> (String)` — `length` defaults to remainder after `from`.
- `suffix(_ max_length: Int) -> (String)`
- `trimmed() -> (String)` — removes leading and trailing whitespace.
- `uppercased() -> (String)`

```kscript
var name = "John"
print("Hello \{name}, over 18: \{18 > 18}")
```

## Array

Ordered, random-access, dynamically growing collection with elements of a single type: `[Element]`.

Gotcha: arrays have **reference semantics** — assignment and passing to functions do NOT copy; mutations affect the original. Use `copy()` for an independent copy.

```kscript
var xs: [Int] = [1, 2, 3]
xs[1] = 42
for i, element in xs.enumerated() {
    print("\{i}, \{element}")
}
```

Arrays can be interpolated into strings (`"\{xs}"` prints `[1, 2, 3]`).

Properties:
- `first: Element?` (get)
- `is_empty: Bool` (get)
- `last: Element?` (get)
- `length: Int` (get)

Methods:
- `all_satisfy(_ predicate: (Element) -> (Bool)) -> (Bool)` — returns `true` on an empty array.
- `append(_ element: Element)` — appends a single element.
- `append(_ other: [Element])` — appends all elements from another array (Kontakt 8.12; replaces `append_all`, removed in 8.12).
- `clear()`
- `contains(_ el: Element) -> (Bool)` — requires `Element` supports `==`.
- `copy() -> ([Element])`
- `enumerated() -> (Enumerated<Element>)` — iterator providing both index and value.
- `filtered(by should_be_included: (Element) -> (Bool)) -> ([Element])`
- `first_match(where: (Element) -> (Bool)) -> (Element?)`
- `index(of element: Element) -> (Int?)` — requires `Element` supports `==`.
- `insert(_ element: Element, at: Int)`
- `joined(separator: String) -> (String)` — requires elements convertible to string.
- `last_index(of element: Element) -> (Int?)` — requires `Element` supports `==`.
- `last_match(where: (Element) -> (Bool)) -> (Element?)`
- `pop_last() -> (Element?)`
- `remove(at index: Int) -> Element`
- `remove_all(where predicate: (Element) -> (Bool))`
- `reverse()` — in place.
- `reversed() -> ([Element])`
- `some_satisfy(_ predicate: (Element) -> (Bool)) -> (Bool)`
- `sort()` — in place, ascending; requires `Element` supports `<`.
- `sorted() -> ([Element])` — requires `Element` supports `<`.
- `sort_by(_ predicate: (Element, Element) -> (Bool))` — in place; predicate returns whether first element sorts before second.
- `sorted_by(_ predicate: (Element, Element) -> (Bool))` — returns new array; predicate returns whether first element sorts before second.
- `subsequence(from: Int, length: Int? = nil) -> [Element]` — `length` defaults to remainder after `from`.

## Map

Collection of key-value pairs: `[Key: Value]`.

- Lookup `map[key]` returns an optional (`Value?`).
- Add/update: `map[key] = value`. Remove: `map[key] = nil` (removing a non-existing key is a no-op).
- Iterable with `for key, value in map { ... }` and interpolatable into strings.
- Gotcha: iteration and print order are **non-deterministic**.
- Supported key types: `Int`, `String`, `Float`, `Bool`, enumerations (since 8.5).

```kscript
var phone_book: [String: String] = ["Malcom": "012345", "Jane": "789078"]
var janes_phone = phone_book["Jane"]
if janes_phone != nil {
    print("Jane: \{janes_phone!}")
}
```

Properties:
- `is_empty: Bool` (get)
- `length: Int` (get)

Methods:
- `clear() -> ()`
- `copy() -> ([Key: Value])` — gotcha: shallow; does not copy values with reference semantics (classes, arrays, etc.).

## Range

Half-open range from lower bound up to, but not including, upper bound. Iterable with `for`.

```kscript skip
class Range {
    lower: Int { get }
    upper: Int { get }

    constructor(_ upper: Int)
    constructor(lower: Int, upper: Int)
}
```

Constructors:
- `(_ upper: Int)` — range from 0 to upper (excluded).
- `(lower: Int, upper: Int)` — range from lower to upper (excluded).

Properties: `lower: Int` (get), `upper: Int` (get).

```kscript
for index in Range(10) { // 0..9
    print("\{index}")
}
var range2 = Range(lower: -10, upper: 10) // -10..9
```

## Color

An RGB color.

Constructors:
- `(red: Int, green: Int, blue: Int)` — 8-bit RGB, channels 0–255.
- `(red: Int, green: Int, blue: Int, alpha: Int)` — 8-bit RGBA, channels 0–255.
- `(_ value: Int)` — composed integer, typically hex literal `0xAARRGGBB`.

Properties: `red: Int` (get), `green: Int` (get), `blue: Int` (get), `alpha: Int` (get).

Methods:
- `opacity(_ value: Float) -> (Color)` — same RGB with given opacity; value must be in `0.0`–`1.0`.
- `to_string() -> (String)`

```kscript
var accent = Color(0xFFFF8800)
var faded = accent.opacity(0.5)
```

## Angle

An angle whose value you can get either as radians or degrees.

```kscript skip
export class Angle {
    degrees: Float { get }
    radians: Float { get }

    constructor(degrees: Float)
    constructor(radians: Float)
}
```

Constructors: `(degrees: Float)`, `(radians: Float)`.
Properties: `degrees: Float` (get), `radians: Float` (get).

```kscript
var turn = Angle(degrees: 90.0)
print("\{turn.radians}")
```
