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

## Known Issues

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

