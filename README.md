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

It is early: the first release is not tagged. The design is RFC-0050 in Byard's repository, and
how a component is written here is in [CONVENTIONS.md](CONVENTIONS.md). What is in `src/`, by
group:

| Group | Views | Needs Byard |
|-------|-------|-------------|
| Actions | `Button` (also the toggle button), `IconButton`, `ButtonGroup`, `MultiButtonGroup` (standard and connected), `SplitButton`, `Fab`, `ExtendedFab`, `FabMenu` | main with RFC-0061 to 0063 (motion in layout, slots, input); `FabMenu` also 0066 |
| Selection | `Switch`, `Checkbox`, `Radio`, `RadioGroup`, `Slider`, `RangeSlider`, `AssistChip`, `FilterChip`, `InputChip`, `SuggestionChip`, `ChipSet` | 0062, 0063 |
| Input | `TextField`, `SearchBar`, `SearchView` | 0064 (multiline, border gap) |
| Containment | `Card`, `ListItem` with `List`, `ListItemIcon`, `ListItemAvatar`, `ListItemImage`, `ListItemText`, `Divider`, `Badge`, `BadgedIcon`, `Carousel` | 0063 |
| Navigation | `NavigationBar`, `NavigationRail`, `Tabs` (with their items), `TopAppBar` | 0065 (context) |
| Toolbars | `BottomAppBar`, `DockedToolbar`, `FloatingToolbar` | 0062 |
| Overlays | `Dialog`, `Menu` (with `MenuGroup`, `MenuItem`, `MenuDivider`), `BottomSheet`, `SideSheet`, `Tooltip`, `Snackbar`, `SnackbarHost` | 0066 (focus scopes, `on enter`/`on leave`, live regions) |
| Progress | `LinearProgressIndicator`, `CircularProgressIndicator`, `LoadingIndicator` | 0064 (`sin`/`cos` in 0066) |
| Layout | `Scaffold` | 0065 |
| Pickers | `DatePicker`, `DatePickerDocked`, `DatePickerModal`, `DateRangePicker`, `DateRangePickerDocked`, `DateRangePickerModal` | 0065, 0066 |
| Icons | `Icon` (Material Symbols Rounded) | icon font axes |

"Needs Byard" is the oldest Byard that has what the group uses: all of it needs Byard's `main`
since 2026-11-14 (RFC-0061 to 0066) until Byard 0.1.0 is tagged, and then 0.1.0. The time picker
is being added.

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
