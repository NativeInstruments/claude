<!-- Condensed from Komplete UI docs (Changelog), keyed by language version -->

# Feature availability by kscript version

Every entry below is a language, standard-library, `ui`, `math`, `path`, `uri` or
`audio_components` change. Host-specific additions are documented by the host's own plugin
(for Kontakt: `kontakt-developer` → `references/kontakt-versions.md`, which also maps
Kontakt releases to the language versions used here).

Versions 1.4, 1.6 and 1.7 were not shipped in a public host release — no entries exist.

| kscript | Notable additions / changes |
|---|---|
| 1.0 | Initial public language. `Map`, `Angle` (replaces Float angles in Rotation/Arc/Canvas), `Range`, `enumerated()`, `clamped`, type deduction for templates and function expressions, unicode code points in strings, named arguments in any order; text modifiers (`LineLimit`, `FontFamily`, `FontSize`, `MultilineTextAlignment`, `TextColor`); Popover fixes (initial visible, anchor following, reload crash). **Breaking:** `Array.filter` → `filtered(by:)`; `subsequence(count:)` → `length:`; `String.replace_all` → `replacing_all`; `String.removing` removed; if-else expression → ternary `a ? b : c`; `Ring` → `Arc`; font size is `Int`; `FontFamily` (class) → `FontFamilyName`; Math `clamp`/`clampf` and `degrees_to_radians`/`radians_to_degrees` removed; exported types must not refer to un-exported types (annotate e.g. `export var main: Component = Main()`); `TapGesture` cancel/up semantics changed |
| 1.1 | `TextInput` component; fixes for declarative `for`/`if` update-before-body and "cycle detected" errors |
| 1.2 | Stable `Drag`/`Drop` modifiers; enumerations as `Map` keys; compile-time improvements |
| 1.3 | `Popover` `visible: Bool` constructor parameter |
| 1.5 | Default function arguments (`fun f(p: Int = 0)`); `\r` escape in strings; `atan2` |
| 1.8 | `letter_spacing` (`Text`, `TextInput`) + `LetterSpacing` modifier; `line_height` (`Text`) + `LineHeight` modifier; faster number parsing; subscript operator (`at`/`assign`) for custom classes |
| 1.9 | Method overloading; `audio_components`, `path` and `uri` packages (`fs` renamed to `path`, `SampleRange` renamed to `VisibleRange`); `Array.append` overloaded to accept `[Element]`, replacing `append_all` |
