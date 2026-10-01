<!-- Condensed from Komplete UI docs (Common Errors, Known Issues, FAQ) -->

# Troubleshooting Reference

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

### Error: `no such parameter with name 'frame'` (Image / NinePatchImage)

Cause: kscript 1.10 replaced `frame`/`frame_count` with `row`/`row_count` (+ `column`/`column_count`) for 2D sprite sheets. Code written for older versions — or a package that wasn't updated — fails once the target language version is 1.10 or newer.

Fix: use `row:`/`row_count:` (a vertical sprite is a single-column sheet). Conversely, `row`/`column` don't exist below 1.10.

### Error: `ambiguous call to method invocation`

Cause: the arguments match several overloads equally well. Since kscript 1.10 the overload needing the fewest implicit conversions wins; a tie (e.g. an `Int` argument for `Float` and `Int?` overloads) is still ambiguous. Fix: pass the exact type (`5.0`), or annotate a variable first.

## Known Issues

- **Font styles split into different families on Windows.** `Roboto-Regular` and `Roboto-Bold` are family `Roboto`, but `Roboto-Medium` is `Roboto Medium`; loading them in one `load_font` call fails. Workaround: one `load_font` call per weight with its bold/italic variants (see `ui-package.md` → `load_font`).

- **Differently composed UTF-8 characters may not compare equal.** Pre-composed vs decomposed forms look identical but can compare unequal in string comparison. (Several string methods were fixed in kscript 1.0; comparison itself remains a known issue.)
- **Spacer size incorrect if modifiers are applied to it.** A modifier applied to a `Spacer` in a stack can alter its size unexpectedly. Workaround: avoid applying modifiers directly on `Spacer`.
- **Indexing the iterated container inside a declarative for loop** (fixed in kscript 1.1): accessing `presets[i]` inside `for i, preset in presets.enumerated() { ... }` can cause out-of-bounds errors when elements are removed. Workaround (pre-1.1): use only the iteration values (`preset`), not `presets[i]`.
- **Same state in a declarative for loop's sequence and its body → "cycle detected" error** (fixed in kscript 1.1): e.g. using `self.offset` both in `presets.subsequence(from: self.offset, length: 1)` and in the loop body. Workaround (pre-1.1): embed the total index/id into the data itself and avoid using the state inside the body:

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

