<!-- Condensed from Komplete UI docs (Changelog) -->

# Kontakt versions & the kscript language version

Kontakt embeds a specific Komplete Script (kscript) language version. Developers working in
Kontakt know the Kontakt version; the language reference in
`kscript-developer:komplete-script` → `references/versions.md` is keyed by **language**
version. Use the mapping below to translate between the two.

## Kontakt release → kscript version

| Kontakt | kscript |
|---------|---------|
| 8.0.0   | 1.0     |
| 8.4.0   | 1.1     |
| 8.5.0   | 1.2     |
| 8.5.1   | 1.3     |
| 8.9.0   | 1.5     |
| 8.11.0  | 1.8     |
| 8.12.0  | 1.9     |

**A Kontakt release not listed inherits the kscript version of the nearest listed release
below it.** So 8.1, 8.2, 8.2.1 and 8.3 → 1.0; 8.6, 8.7 and 8.8 → 1.3; 8.10 → 1.5. kscript
1.4, 1.6 and 1.7 were never shipped in a public Kontakt release.

## Determining the ceiling

`kontaktTargetVersion` in `Resources/info/library.json` declares the oldest Kontakt version
the instrument must run on. It gates the whole feature set:

1. Read `kontaktTargetVersion`.
2. Map it to a kscript version with the table above.
3. Treat that kscript version as the ceiling for everything in the language references, and
   the Kontakt version itself as the ceiling for the Kontakt-only features below.

Never use a feature newer than the target.

## Kontakt-specific feature availability

Language, stdlib and `ui` changes are **not** listed here — see
`kscript-developer:komplete-script` → `references/versions.md`.

| Kontakt | Kontakt-specific additions / changes |
|---|---|
| 8.0 (beta) | Komplete UI public beta; `komplete_scripts` folder in the resource container; UI loaded via `load_komplete_ui("module_name")` from KSP |
| 8.1 (beta) | `KSPTextEdit`, `KSPLabel` |
| 8.2 | Kontakt Controls `XYPad` and `Stepper`; `TogglePolicy` → `TriggerPolicy`. **Breaking:** `load_komplete_ui()` removed — the main module is selected in *Instrument Options* instead |
| 8.3 | `KSPTable.length` |
| 8.4 | `text` property on `KSPButton`/`KSPKnob`/`KSPSwitch`/`KSPValueEdit`; `KSPTextEdit` `text` setter |
| 8.8 | `KSPMenu.entries`; `Help` modifier (Info Pane help text, `info_hints.json`) |
| 8.12 | `kontakt` module gains `Instrument`/`instrument`, `Group`, `GroupList`, `GroupListIterator`, `Zone`, `ZoneList`, `ZoneListIterator`, `Sample`, `load_sample`, `library_path` |
