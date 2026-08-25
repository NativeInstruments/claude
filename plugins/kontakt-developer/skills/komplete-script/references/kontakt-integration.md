<!-- Condensed from Komplete UI docs (Language Reference, Kontakt integration) -->

# Kontakt Integration Pattern

Language syntax is in `kscript-developer:komplete-script` → `references/language.md`. This
file covers only how a kscript UI connects to the Kontakt Engine through KSP.

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

## Rules

- **KSP connections must be declared globally.** They are fixed at load time and do not
  belong inside a component.
- Per-control classes and their full property sets: `references/kontakt-package.md`.
- Ready-made KSP-connected controls: `references/kontakt-controls.md`.
