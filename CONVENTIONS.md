# How a component is written here

Read this, then `src/button.byd`. Every other component is Button again with different parts.
When this file and the code disagree, the code was run and this was not: fix this file.

The package follows RFC-0050 (in Byard's repository). Material values are the public Material 3
Expressive specification. A value you are not sure of is not invented: write down what you used
and that you were unsure, in your notes.

## Layout

```
byard.toml          manifest, theme (colours, type, shape), fonts
src/tokens.byd      private constants every control shares (state opacities, disabled, focus ring)
src/enums.byd       enums several components share (ButtonSize, ButtonShape)
src/icon.byd        public Icon
src/<family>.byd    private measurements shared by a family (button_family.byd)
src/<component>.byd one public component per file, with its own enums and private helpers
gallery/            an app that depends on the package by path: an index and a page per component
gallery/tests/      `byard test` scenarios, one file per component
```

One public component per file, named after the view in snake case (`IconButton` is
`icon_button.byd`). `lib.byd` has no code; a package is one namespace.

## Naming

- Views are Material's names without a prefix (`Button`, `NavigationBar`); the consumer's
  `use material as m` is the prefix.
- An enum is the component's name plus what it varies (`ButtonVariant`); cases are lower case
  (`.filled`, `.tonal`). Never take a variant as a `Str`.
- Parameters are `snake_case`: what it shows (`label`, `icon`), variant, size, shape, the Bools
  (`toggle`, `selected`, `disabled`), then callbacks last (`on_tap: Fn() = {}`). A view takes its
  action by name; `m.Button("x") => f()` is an error.
- "Selected" and "checked" are plain Bools plus a callback. A control that owns a value an app
  edits (`Switch`, `TextField`, `Slider`) takes a binding: `bind on: Bool`.
- Private things are `private`. Constants are camelCase. Per-size numbers are a private `fn` over
  the size enum, a chain of `?:`, with Material's table in a comment above it.

## The theme

Colours, type, shape and motion come from `inject Theme as t`, never a literal.

| What | Write |
|------|-------|
| colour | `t.primary`, `t.onSurface.alpha(0.12)` |
| text | `typo: t.labelLarge` (a `?:` of tokens is fine) |
| corner, size, shape morph | `with t.springSpatialFast` (may overshoot) |
| colour, opacity, shadow | `with t.springEffectsFast` (does not overshoot) |
| icon fill | the `Icon` view animates it with `t.springEffectsDefault` |

A property that changes with state has a `with`; nothing animates while the control is still.
`.alpha(x)` replaces the colour's alpha, it does not multiply. Sizes that belong to the component
are numbers in a private `fn`, not theme tokens.

## A control

One element carries everything: the look, the role, the tap and the states. No `var`, no pointer
or focus handler, no wrapper view.

```byld
Row #[
    bg: container with t.springEffectsFast,
    radius: r with t.springSpatialFast,
    border: line,
    border_width: outlined ? lineWidth : 0,
    shadow: restShadow,
    height: h,
    p: (0, buttonPadding(size)),
    gap: buttonGap(size),
    align: center,
    justify: center,
    a11y_role: toggle ? toggle_button : button,
    a11y_checked: selected,
    disabled: disabled,
    tap => on_tap(),
] {
    on hover { overlay: ink.alpha(stateHover) with t.springEffectsFast, shadow: hoverShadow }
    on focus_visible {
        overlay: ink.alpha(stateFocus) with t.springEffectsFast
        outline: t.secondary
        outline_width: focusRingWidth
        outline_offset: focusRingOffset
    }
    on pressed {
        overlay: ink.alpha(statePress) with t.springEffectsFast
        radius: buttonPressedRadius(size) with t.springSpatialFast
    }

    // children: Icon, Text
}
```

- **State layer**: `overlay:` in `on hover`, `on focus_visible`, `on pressed`, the content colour
  at `stateHover`, `stateFocus`, `statePress` (`tokens.byd`). Write them in that order: on a tie
  the later block wins, and a press outranks focus outranks hover. A disabled control is never
  hovered, pressed or focused, so none of them applies to it and no guard is needed.
- **Focus ring**: `outline` in `on focus_visible`, keyboard focus only (a click draws none), with
  `focusRingWidth` and `focusRingOffset`. It sits outside the shape, follows `radius` and takes
  no room. An ancestor's `Clip` or `ScrollView` cuts it, so leave a gap.
- **Press morph**: `radius` in `on pressed` with a spatial spring. The state's `with` also
  animates the way back, with no `with` needed on the resting value for that.
- **Held keys**: Space or Enter held on a focused control shows `on pressed` too.
- **Role from a value**: `a11y_role: toggle ? toggle_button : button`. One root serves both.
- **Elevation**: the presets `"sm"` and `"md"` stand for levels 1 and 2. A `let` holding a preset
  or `()` is fine, as is a `?:`.
- **Disabled**: `disabled: disabled` on the root. The dump shows `[focusable disabled]`, a tap does
  nothing, Tab skips it and no state block applies (`on disabled` is the only one that does). The disabled colours are `disabledContainer` and `disabledContent`
  of `on_surface`.
- **Toggle icon**: `Icon(icon, fill: on ? 1.0 : 0.0, ...)`, filled when selected, outline when not;
  `Icon` animates the change.
- Put the control on a `Row` when it lays out children in a row, a `Box` when it has one child
  (IconButton). `shrink: 0` on a fixed-size control, `wrap: false` on its label.
- Precedence: `?:` binds tighter than `||`, so write `(a || b) ? x : y`.

## Accessibility checklist

From `byard shot --a11y out.txt`, and enforced by `byard test`:

- [ ] a role that says what it is (`button`, `toggle_button`, `switch`, `checkbox`, `radio`, `tab`, `slider`)
- [ ] a name: its text, or `a11y_label` from a required `label` parameter for an icon control
      (`Icon` hides itself from readers unless given a `label`)
- [ ] state: `a11y_checked`, `a11y_selected`, `a11y_expanded`, `a11y_value` as the role needs
- [ ] `disabled: disabled`, and a disabled one does not act
- [ ] Tab reaches it in reading order; Space and Enter press it; arrows where Material uses them
- [ ] the ring shows after `--key Tab` and not after `--tap`
- [ ] nothing animates when nothing changes

## Value checklist

- [ ] every variant and size Material specifies, light and dark
- [ ] hover, press, focus and disabled draw as specified
- [ ] colours per variant and state from the spec's token table, listed in a comment or your notes
- [ ] a value you were unsure of is listed in your notes as such

## The gallery

`gallery/src/main.byd` is an index of the pages (a `var page` and a `when` chain); each page is a
view of its own, so `byard shot . --view ButtonPage` draws it. To add a component:

1. A case in `enum Page`, a button for it in the index, a branch in the chain.
2. `gallery/src/pages/<component>.byd` with `use material as m`, a view `<Component>Page` made of
   `Section(title) { .. }` and `Line(caption) { .. }` (`parts.byd`): every variant and size as a
   grid, then square, with icon, toggle (off, then on) and disabled. A toggle row owns its `var`s.
3. In the same file, `<Component>Scenarios`: a few controls with names of their own (`Save`,
   `Star`, `Archive`), because a page repeats names and `byard test` taps by name.
4. `gallery/tests/<component>.toml` on that view: names and roles in the tree, a tap acts, a toggle
   flips, Tab reaches the first control, Space presses it, a disabled one does not act.
5. Look at it, from `gallery/` (`byard get` first):

```sh
byard shot . --view ButtonPage --size 1000x2900 -o light.png --a11y a11y.txt
byard shot . --view ButtonPage --size 1000x2900 --dark -o dark.png
byard shot . --view ButtonPage --size 1000x2900 --hold 347,127 -o hold.png
byard shot . --view ButtonPage --size 1000x2900 --hover 347,431 -o hover.png
byard shot . --view ButtonPage --size 1000x2900 --key Tab -o focus.png
```

   Take coordinates from the dump (`@ x,y WxH`: the centre is `x + W/2`, `y + H/2`). Look at every
   picture; crop with `magick shot.png -crop 300x200+550+160 +repage c.png`. Pictures are not
   committed.
6. `byard fmt --check`, `byard check` in the package and `gallery/`, and `byard test` in
   `gallery/`, all clean.

## Worked example: Button

`src/button.byd`, top to bottom:

1. `enum ButtonVariant` is its own; `ButtonSize` and `ButtonShape` are shared (`enums.byd`).
2. `View Button(label, variant: ButtonVariant = .filled, ...)`: enum defaults in the short form.
3. `container`, `ink`: one `?:` chain per variant, `disabled` first, the toggle's selected colours
   inline. `outlined`, `line`, `lineWidth` for the outline variant.
4. `r`: the resting corner (`square` flips when a toggle is on, `h / 2` is fully round).
5. `restShadow`, `hoverShadow`: the elevation.
6. One `Row` with the attributes, the three state blocks and the children above; the label's `typo`
   is a `?:` over the sizes.

`IconButton` is the same with a `Box` of fixed width and one `Icon`: copy it for a single-icon
control, Button when there is a label.
