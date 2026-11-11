# Byard Material

Material 3 Expressive components for [Byard](https://github.com/Briany4717/byard), written in
Byld and shipped as a Byard package.

```toml
[dependencies]
material = { git = "https://github.com/Briany4717/byard-material", tag = "v0.1.0" }
```

```byld
use material as m

View Main() {
    var starred = false

    Row #[gap: 12, align: center] {
        m.Button("Save", icon: "check", on_tap: { save() })
        m.IconButton("star", label: "Star", variant: .tonal, toggle: true, selected: starred,
            on_tap: { starred = !starred })
    }
}
```

and, in the app's `byard.toml`, `[theme] extends = "material"` so the colours, type and shapes
are Material's (a `seed` there recolours all of it).

It is early. The design is RFC-0050 in Byard's repository. So far there are `Button`,
`IconButton`, `Icon`, the four chips (`AssistChip`, `FilterChip`, `InputChip`, `SuggestionChip`),
`Switch`, `Checkbox`, `Radio`, `RadioGroup`, `Slider` and `RangeSlider`; the rest of the
components are on their way. How a component is
written here is in [CONVENTIONS.md](CONVENTIONS.md).

This package is also how Byard's package system gets tested for real: it lives in its own
repository and goes through `byard add`, `byard get`, the lockfile, `byard publish` and
`byard bundle` like anyone else's package would.

## The gallery

`gallery/` is an app that uses the package by path and shows every component.

```sh
cd gallery
byard get
byard dev
```

## Licence

MIT or Apache-2.0, at your option.

This package implements the public Material 3 guidelines. It is not made by, endorsed by or
affiliated with Google.

## Fonts

Two fonts ship in `fonts/`, each with its licence:

- **Material Symbols Rounded**, Google's icon font, under the Apache License 2.0
  (`fonts/LICENSE-MaterialSymbols`). It is declared as an icon font, so `byard bundle` keeps
  only the icons an app uses.
- **Roboto Flex**, the text face, under the SIL Open Font License 1.1
  (`fonts/OFL-RobotoFlex.txt`).
