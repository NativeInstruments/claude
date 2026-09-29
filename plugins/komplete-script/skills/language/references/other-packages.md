<!-- Condensed from Komplete UI docs (Math, URI, Path, Audio Components packages) -->

# Math

```kscript
import * from math
```

## Constants

- `e: Float` — mathematical constant e (2.718281828459)
- `max_integer: Int` — maximum signed 64-bit Int (9223372036854775807)
- `min_integer: Int` — minimum signed 64-bit Int (-9223372036854775808)
- `pi: Float` — 3.1415926535898

## Functions

#### abs(_ value: Int) -> (Int)

```kscript
var a = abs(-12) // a equals 12
```

#### absf(_ value: Float) -> (Float)

```kscript
var f = absf(-34.5) // f equals 34.5
```

#### acos(_ value: Float) -> (Float)

Arccosine, radians.

```kscript
var arccosine = acos(1.0) // arccosine equals 0.0
```

#### asin(_ value: Float) -> (Float)

Arcsine, radians.

```kscript
var arcsine = asin(0.0) // arcsine equals 0.0
```

#### atan(_ value: Float) -> (Float)

Arctangent, radians.

```kscript
var arctangent = atan(1.5) // arctangent equals 0.98279372324733
```

#### atan2(y: Float, x: Float) -> (Float)

Arc tangent of y / x using argument signs for quadrant. Since kscript 1.5.

```kscript
var arctangent2 = atan2(y: 7, x: 0) // arctangent2 equals 1.5708
```

#### cos(_ value: Float) -> (Float)

Cosine, radians.

```kscript
var cosine = cos(0.0) // cosine equals 1.0
```

#### exp(_ value: Float) -> (Float)

e to the power of value.

```kscript
print("\{exp(1.0)}") // Prints 2.718281828459
```

#### log(_ value: Float) -> (Float)

Natural logarithm.

```kscript
var logarithm = log(1.0) // logarithm equals 0.0
```

#### max(_ value1: Int, _ value2: Int) -> (Int)

```kscript
var maximum = max(3, 12) // maximum equals 12
```

#### maxf(_ value1: Float, _ value2: Float) -> (Float)

```kscript
var maximumf = maxf(3.14, 12.7) // maximumf equals 12.7
```

#### min(_ value1: Int, _ value2: Int) -> (Int)

```kscript
var minimum = min(3, 12) // minimum equals 3
```

#### minf(_ value1: Float, _ value2: Float) -> (Float)

```kscript
var minimumf = minf(3.14, 12.7) // minimumf equals 3.14
```

#### random(from: Int, to: Int) -> (Int)

Uniform random integer between the given values.

```kscript
var rand = random(from: -10, to: 1000)
```

#### randomf(from: Float, to: Float) -> (Float)

Uniform random float between the given values.

```kscript
var randf = randomf(from: 45.4, to: 45.9)
```

#### sin(_ value: Float) -> (Float)

Sine, radians.

```kscript
var sine = sin(0.0) // sine equals 0.0
```

#### sqrt(_ value: Float) -> Float

```kscript
var radical = sqrt(4.0) // radical equals 2.0
```

# URI

kscript 1.9.

```kscript
import * from uri
```

## class URI

Represents a Uniform Resource Identifier. No constructor — obtain instances via `parse_uri` / `parse_uri_list`.

```kscript
export class URI {
    has_fragment: Bool { get }
    has_port: Bool { get }
    has_query: Bool { get }
    has_scheme: Bool { get }
    scheme: String { get }
    host: String { get }
    port: String { get }
    path: String { get }
    query: String { get }
    fragment: String { get }

    equal(to other: URI) -> Bool
    to_string() -> String
}
```

```kscript
import { URI } from uri
```

Properties (all get-only): `has_fragment`, `has_port`, `has_query`, `has_scheme` (Bool — component presence); `scheme` (e.g. `https`, `file`), `host`, `port` (as String), `path`, `query` (without leading `?`), `fragment` (without leading `#`).

Methods:
- `equal(to other: URI) -> (Bool)` — equality comparison.
- `to_string() -> (String)` — full URI as string.

## fun parse_uri(_ uri: String) -> (URI?)

Parses a URI string; returns `nil` if invalid.

```kscript
import { parse_uri } from uri
```

Parameter: `_ uri: String` (required) — e.g. `https://www.example.com/path?query=1#fragment`.

## fun parse_uri_list(_ uri_list: String) -> ([URI])

Parses a `text/uri-list` formatted string, returns all valid URIs. Lines starting with `#` are comments; empty lines ignored. Entries separated by `\r\n` (OS drag data conforms).

```kscript
import { parse_uri_list } from uri
```

Example — file drop from Finder/Explorer:

```kscript
import { Rectangle, Frame, Overlay, Drop, Text } from ui
import { parse_uri_list } from uri

export component Main {
    dropped_paths: String = "Drop files here"
    contains_drop: Bool = false

    Rectangle(color: self.contains_drop ? Color(0xFF50A0C0) : Color(0xFF80C0D0)) with {
        Frame(width: 400, height: 80)
        Overlay {
            Text(self.dropped_paths)
        }
        Drop(
            of: ["text/uri-list"],
            enter: fun (event) { self.contains_drop = true },
            exit: fun () { self.contains_drop = false },
            perform: fun (event, action) {
                var uris = parse_uri_list(event.data(format: "text/uri-list"))
                var paths: [String] = []
                for uri in uris {
                    paths.append(uri.path)
                }
                self.dropped_paths = paths.joined(separator: "\n")
                self.contains_drop = false
                return true
            },
        )
    }
}

export var main = Main()
```

# Path

kscript 1.9. Package renamed `fs` → `path`.

```kscript
import * from path
```

## class Path

Represents a filesystem path.

```kscript
export class Path {
    parent: Path { get }
    root_name: Path { get }
    root_directory: Path { get }
    root: Path { get }
    filename: Path { get }
    stem: Path { get }
    extension: Path { get }
    has_parent: Bool { get }
    has_root_name: Bool { get }
    has_root: Bool { get }
    has_filename: Bool { get }
    has_stem: Bool { get }
    has_extension: Bool { get }
    is_absolute: Bool { get }
    is_relative: Bool { get }
    is_empty: Bool { get }

    constructor(_ path: String)

    appending(_ path: String) -> Path
    appending(_ other: Path) -> Path
    appending(_ segments: [Path]) -> Path
    removing_filename() -> Path
    replacing_filename(_ filename: Path) -> Path
    replacing_extension(_ extension: Path) -> Path
    relative(to base: Path) -> Path
    equal(to other: Path) -> Bool
    to_string() -> String
    to_uri() -> URI?
}
```

```kscript
import { Path } from path
```

Constructor: `_ path: String` (required) — string representation of the path.

Properties (all get-only; component getters return empty path if absent):
- `parent: Path` — parent path component.
- `root_name: Path` — drive identifier on Windows (e.g. `C:`); always empty on POSIX.
- `root_directory: Path` — root directory separator (e.g. `/` on POSIX).
- `root: Path` — root name combined with root directory.
- `filename: Path` — last path component.
- `stem: Path` — filename without extension.
- `extension: Path` — file extension including leading dot (e.g. `.wav`).
- `has_parent`, `has_root_name`, `has_root`, `has_filename`, `has_stem`, `has_extension`, `is_absolute`, `is_relative`, `is_empty`: Bool.

Methods (`appending` is overloaded, replacing the old `appending_all`/`appending_path` — kscript 1.9):
- `appending(_ path: String) -> (Path)` — append string as new path component.
- `appending(_ other: Path) -> (Path)` — append `other`'s components.
- `appending(_ segments: [Path]) -> (Path)` — append all segments from the array in order.
- `removing_filename() -> (Path)` — filename component removed.
- `replacing_filename(_ filename: Path) -> (Path)`
- `replacing_extension(_ extension: Path) -> (Path)`
- `relative(to base: Path) -> (Path)` — path expressed relative to `base`.
- `equal(to other: Path) -> (Bool)` — equality, ignoring path separator differences.
- `to_string() -> (String)` — UTF-8 string representation.
- `to_uri() -> (URI?)` — file URI of format `file://<path>`; `nil` for an empty path.

## fun path_from_uri(_ uri: URI) -> (Path?)

Creates a `Path` from a file URI. Returns `nil` if the URI cannot be converted to a path.

```kscript
import { path_from_uri } from path
```

Parameter: `_ uri: URI` (required).

# Audio Components

kscript 1.9.

```kscript
import * from audio_components
```

## class VisibleRange

Half-open range of samples within an audio sample: [begin, end). Floating point for sub-sample precision. Renamed from `SampleRange` in kscript 1.9.

```kscript
export class VisibleRange {
    begin: Float
    end: Float
}
```

```kscript
import { VisibleRange } from audio_components
```

Constructor: `begin: Float` (required) — starting sample index; `end: Float` (required) — ending sample index.

Properties: `begin: Float` (get/set), `end: Float` (get/set).

## component Waveform

Displays a `Sample`. Layout behavior: fills the available area.

```kscript
export component Waveform {
    @property sample: Sample
    @property channel: Int = 0
    @property range: VisibleRange? = nil
    @property color: Color = 0xFF000000
}
```

Constructor parameters:
- `sample: Sample` (required) — the sample to display.
- `channel: Int = 0` — channel of the sample to display.
- `range: VisibleRange? = nil` — range of samples to display; `nil` displays the entire sample.
- `color: Color = 0xFF000000` — fill color.

```kscript
import { Waveform } from audio_components

// `my_sample` stands in for a concrete Sample obtained from the host — the
// loading function is host API, not part of `audio_components`.
export var main = Waveform(sample: my_sample)
```

## interface Sample

Audio sample displayable by `Waveform`. This is an **interface only** — it has no constructor. The concrete implementation and the function that loads it are supplied by the host (in Kontakt: `kontakt.Sample` via `load_sample` — see that plugin's `kontakt-package.md`).

```kscript
export interface Sample {
    num_channels: Int { get }
    num_frames: Int { get }
}
```

```kscript
import { Sample } from audio_components
```

Properties: `num_channels: Int` (get) — number of channels; `num_frames: Int` (get) — frames (samples per channel).
