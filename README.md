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
    var wifi = true
    m.Switch(bind: wifi)
}
```

It is early. The design is RFC-0050 in Byard's repository. Nothing is released yet, and the
components above don't exist yet either.

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
