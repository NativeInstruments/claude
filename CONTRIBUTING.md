# Contributing

## How updates reach users

Plugins in this marketplace are distributed via git
(`github.com/NativeInstruments/claude.git`). None of the `plugin.json` files
declare a `version` field. This is intentional.

When `version` is **omitted** and a plugin is distributed via git, Claude Code
uses the commit SHA as the version — so **every commit automatically counts as a
new version**. Users get your change on their next
`/plugin marketplace update native-instruments`; no manual step required.

If you **add** a `version` field, that changes: users then only receive updates
when you bump that field. Forgetting to bump it means changes silently never
reach anyone. Avoid adding `version` unless you deliberately want manual,
controlled releases — and if you do, bumping it becomes mandatory for every
user-visible change.

> `/plugin marketplace update` refreshes the marketplace catalog. Installed
> plugins update from that refreshed catalog.

## Adding or changing a plugin

1. Edit files under `plugins/<plugin-name>/`.
2. Document any new command or skill in that plugin's `README.md` (args,
   defaults, when it triggers).
3. Register new plugins in `.claude-plugin/marketplace.json`.
4. Commit. That's the release.

## Plugins that depend on other plugins

`kontakt-developer` builds on `kscript-developer`: the Kontakt skill covers only the Kontakt
layer and loads the language skill for kscript itself. There is no dependency field in
`plugin.json`, and — because no plugin declares a `version` — no way to pin one either. So a
cross-plugin dependency is expressed two ways:

1. Documented in both `README.md` files as a prerequisite.
2. Asserted at skill load time: the depending skill instructs loading the other skill first,
   and tells the user which `/plugin install` to run if it is missing.

Keep both in sync when adding a dependency, and never duplicate reference material across
plugins — one source of truth, referenced by name.

## Documenting commands

Command args live in the command file's frontmatter (`argument-hint`) and are
only visible there or via the inline hint after typing the command. Always
mirror them in the plugin `README.md` so they're discoverable without reading
source.
