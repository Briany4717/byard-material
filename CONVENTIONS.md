# How a component is written here

Read this, then read `src/button.byd` and `src/icon_button.byd`. Every other component is
Button again with different parts. When this file and the code disagree, the code was run and
this was not: fix this file.

The package follows RFC-0050 (in Byard's repository). Material values are the public Material 3
Expressive specification. A value you are not sure of is not invented: write down what you used
and that you were unsure, in your notes.

## Layout of the package

```
byard.toml          manifest, theme (colours, type, shape), fonts. Rarely touched.
src/tokens.byd      private constants shared by every control (state opacities, focus ring)
src/state_layer.byd private StateLayer and FocusRing views
src/enums.byd       enums that several components share (ButtonSize, ButtonShape)
src/icon.byd        public Icon
src/<family>.byd    private measurements shared by a family (button_family.byd)
src/<component>.byd one public component per file, with its own enums and private helpers
gallery/            an app that depends on the package by path: one page per component
```

One public component per file, named after the view in snake case: `Button` is `button.byd`,
`IconButton` is `icon_button.byd`. A file holds the component, its own enums, the private views
it is built from (`ButtonFace`, `ButtonLabel`) and its private functions. Anything two
components need goes in a file of its own and is `private` unless an app needs it.

`lib.byd` has no code. A package is one namespace: every file sees every other file's names.

## Naming

- Views are Material's names without a prefix: `Button`, `IconButton`, `NavigationBar`. The
  consumer's `use material as m` is the prefix. A view named like a built-in (`Button`,
  `TextField`, `Checkbox`, `Slider`) is fine in a package; inside the package a bare `Button`
  is still the built-in.
- An enum is the component's name plus what it varies: `ButtonVariant`, `IconButtonWidth`.
  Cases are lower case, as Material says them: `.filled`, `.tonal`, `.narrow`.
  A caller writes `variant: .tonal`. Never take a variant as a `Str`: a typo is silent.
- Parameters are `snake_case`. The first is what the component shows (`label`, `icon`), then
  variant, size, shape, then the Bools (`toggle`, `selected`, `disabled`), then callbacks last:
  `on_tap: Fn() = {}`. A view takes its action by name; `m.Button("x") => f()` is an error.
- Values that Material calls "selected" or "checked" are plain Bools plus a callback
  (`selected: starred, on_tap: { starred = !starred }`). A control that owns a value an app edits
  (`Switch`, `TextField`, `Slider`) takes it as a binding: `View Switch(bind on: Bool, ...)`, used
  as `m.Switch(bind: wifi)`.
- Private names: `private View`, `private fn`, `private let`, `private enum`. Constants are
  camelCase (`stateHover`). Per-size numbers are a private `fn` over the size enum
  (`buttonHeight(size)`), written as a chain of `?:` with the Material table above it in a comment.

## The rule about the theme

Colours, type, shape and motion come from `inject Theme as t` and nowhere else. No hex literal,
no `radius: 20` that means "round" (use half the height), no `anim.spring(stiffness: ..)`.

| What | Write | Never |
|------|-------|-------|
| colour | `t.primary`, `t.onSurface.alpha(0.12)` | `0x6750A4` |
| text | `typo: t.labelLarge` (see the warning below) | `size: 14, weight: 500` |
| corner | `radius: r with t.springSpatialFast` | a literal when a token exists |
| position, size, shape morph | `with t.springSpatialFast` | `with anim.spring()` |
| colour, opacity, alpha | `with t.springEffectsFast` | |
| icon fill | `with t.springEffectsDefault` | |

Spatial springs may overshoot and are for things that move or change shape; effects springs are
for colour and opacity and do not overshoot. Put a `with` on every property that changes with state;
a value that never changes needs none, and nothing animates while the control is still.

Sizes that belong to the component (a button is 40 tall) are numbers in a private `fn`, from
Material's specification. They are not theme tokens.

## The shared pieces

Use these, do not copy them.

| Piece | Use it for |
|-------|------------|
| `stateHover`, `stateFocus`, `statePress`, `stateDragged` (`tokens.byd`) | the state-layer opacities: 8%, 10%, 10%, 16% |
| `disabledContent`, `disabledContainer` | 38% of `on_surface` for content, 12% for the container |
| `StateLayer` | the tint a state puts over the container. Wrap the control's content in it |
| `FocusRing` | the 3 dp `secondary` ring, 2 dp outside the control. Wrap the whole control in it |
| `Icon` | every icon, with `color:` the content colour |

A control is built from the outside in, and this order is the same for every one:

```
FocusRing                 ring, shown while the keyboard has focus
  interactive Box         container, shape, border, shadow, a11y role and name, tap, handlers
    StateLayer            hover / focus / press tint, same radius as the container
      content             icon, label
```

The interactive Box is the only element with handlers, so a press lands on it. `StateLayer` and
`FocusRing` have none and never intercept one.

### The pointer and focus handlers

Every interactive Box carries the same handlers and three `var`s. Copy this block:

```byld
var hov = false
var down = false
var kbd = false          // "focus_visible": focus that the keyboard gave

Box #[..chrome, a11y_role: button, disabled: disabled,
    tap => on_tap(),
    pointer_enter => hov = true,
    pointer_exit => {
        hov = false
        down = false
    },
    pointer_down => down = true,
    pointer_up => down = false,
    focus => kbd = !down,
    blur => kbd = false] {
    ...
}
```

`kbd` is Byard's `focus_visible`: the engine's own state is only readable by the element it is
on (`on focus_visible` in a style), and the ring has to be a sibling outside the control, so the
handlers compute the same thing: focus that arrives while no pointer press is down came from the
keyboard (or a screen reader). A click focuses without a ring. Test it: `--tap` leaves no ring and
`--key Tab` draws one.

`hov`, `down` and `kbd` then feed three places: the shape (`down` morphs the corners), the
`StateLayer` (`hovered:`, `pressed:`, `focused: kbd`) and `FocusRing(visible: kbd)`.

### Why two roots

`a11y_role` is a keyword, not a value, so a control that has two roles has two roots. Button and
IconButton are `when toggle { Box #[a11y_role: toggle_button, a11y_checked: selected, ..] }
else { Box #[a11y_role: button, ..] }` around the same face. Keep the shared look in a
`let chrome = style { .. }` and the shared inside in a private `...Face` view so the two roots
differ only by their role lines.

### Layout notes that cost time

- A bare `Column` child stretches to the column's width. A control in a `Column` that should
  hug its content is placed in `Column #[align: start]` by the app; the component sets no
  `align_self`, so it also centres in a `Row #[align: center]`.
- `FocusRing` has negative margin to reach outside without taking room, so a ring is drawn over a
  neighbour's gap, never over its pixels, and a `Clip` or `ScrollView` edge cuts it.
- `StateLayer` hugs its content, so give the content the control's size (a fixed `width` and
  `height` on the box inside) when the control has a fixed size (IconButton).
- A `Text` or `Icon` that must not be squeezed in a `Row` takes `shrink: 0`.

## Things that do not work yet (and the way around)

Each of these is also in the notes of the foundation branch with its smallest reproduction.

- `typo:` chosen with `?:` loses the style's family and weight. Choose the token with `when` /
  `else when`, one `Text` per case (`ButtonLabel`).
- A `shadow:` that is a list of tuples held in a `let`, or picked with `?:`, is not drawn; a
  `with` on a tuple shadow is not drawn either. Use the presets, `"sm"` and `"md"`, which a
  `?:` can pick (`shadow: lifted ? "md" : ()`).
- A transparent `Box` with only a `border` draws nothing: give it `bg: t.surface.alpha(0.0)`.
- Spaces matter in ternary chains: `a || b ? x : y` reads `a || (b ? x : y)`. Parenthesize.

## Accessibility and keyboard checklist

A component is not done until each line is true, looking at `byard shot --a11y out.txt`:

- [ ] It has an `a11y_role` that says what it is (`button`, `toggle_button`, `switch`, `checkbox`,
      `radio`, `tab`, `slider`, ...). Never leave a tapped Box without one.
- [ ] It has a name. Text inside it names it. A control with no text (an icon button) gets
      `a11y_label` from a required `label` parameter. The icon inside is `a11y_hidden`
      (`Icon` is, unless given a `label`).
- [ ] State is announced: `a11y_checked` for switch, checkbox, radio and toggle button;
      `a11y_selected` for tabs and list items; `a11y_expanded` for what expands;
      `a11y_value` for sliders and fields.
- [ ] `disabled: disabled` on the root: the dump shows `[focusable disabled]` and Tab skips it.
- [ ] Tab reaches it, in reading order. Space and Enter press it (the engine does it for a
      composed control with a role and a `tap`). `--key Tab --key Space` runs `on_tap`.
- [ ] Arrow keys where Material uses them: a `radio_group` or `tab_list` wraps its members and the
      engine walks them; a slider steps with arrows.
- [ ] The ring shows after `--key Tab` and not after `--tap`.
- [ ] Nothing animates when nothing changes: no looping animation unless the component is a
      progress indicator that is on screen.

## Value checklist, per component

- [ ] every variant and size Material specifies, light and dark;
- [ ] hover, press, focus and disabled draw as specified; the press morph uses
      `t.springSpatialFast`;
- [ ] colours for each variant and state taken from the spec's token table, listed in a comment
      or your notes;
- [ ] a value you were not sure of is listed in your notes as such.

## The gallery

`gallery/src/main.byd` is an index: `enum Page`, a `let page = Page.button` at the top, and a
`when`/`else when` chain showing that page. Change the constant to see another page, until
`byard shot --view` exists.

To add a component's page:

1. Add its case to `enum Page`, its title to the header text, and its branch to the chain in
   `Main`.
2. Create `gallery/src/pages/<component>.byd` with `use material as m` and a view
   `<Component>Page`. Sections come from `Section(title) { .. }` and rows from
   `Line(caption) { .. }` (`gallery/src/parts.byd`). Show every variant, size and state in a grid:
   variants as rows, sizes or states as columns; then square/round, with icon, toggle
   (unselected then selected), and disabled. A toggle row owns its `var`s so it can be tapped.
3. Look at it:

```sh
cd gallery
byard get                  # once, and after changing the manifest
byard fmt .. && byard check .
byard shot . --size 1100x3000 -o button-light.png --a11y button-a11y.txt
byard shot . --size 1100x3000 --dark -o button-dark.png
byard shot . --size 1100x3000 --hold 246,185 --at 200 -o button-hold.png   # held press
byard shot . --size 1100x3000 --hover 347,185 -o button-hover.png          # hover
byard shot . --size 1100x3000 --key Tab --key Tab -o button-focus.png      # keyboard focus
```

   Take the coordinates from the `--a11y` dump (`@ x,y WxH`: the centre is `x + W/2`, `y + H/2`).
   Look at every picture and compare with Material's spec before committing; crop with
   ImageMagick to read the details (`magick shot.png -crop 1300x560+200+220 +repage c.png`).
   Pictures are not committed (`*.png` is ignored); put what you looked at in your notes.
4. `byard fmt --check` and `byard check` in the package and in `gallery/`, both clean, and
   `byard get` in `gallery/` before committing (it only rewrites the lock when the manifest
   changed).

## Worked example: Button

`src/button.byd`, top to bottom:

1. `enum ButtonVariant` is its own; `ButtonSize` and `ButtonShape` are shared (`enums.byd`).
2. `View Button(label, variant: ButtonVariant = .filled, size: ButtonSize = .s, ...)`: enum
   defaults use the short form.
3. The three `var`s and the handler block above.
4. `container`, `ink`: one `?:` chain per variant, with the toggle's selected colours inline.
   `disabled` comes first in each chain, with `disabledContainer` and `disabledContent`.
5. `r`: the corner radius. `down` gives the pressed radius, otherwise the resting shape
   (`square` flips when a toggle is on, `h / 2` is fully round).
6. `chrome`: a `style` with every changing property and its `with t.spring...`.
7. `FocusRing(radius: r, visible: kbd) { when toggle { ..toggle_button root.. } else { ..button root.. } }`.
8. `ButtonFace`: `StateLayer` around a `Row` with the optional `Icon` and `ButtonLabel`.
9. `ButtonLabel`: the label's `typo`, one branch per size.

`IconButton` is the same nine steps with different colours and a fixed width: copy from it when
the component is a single icon, from `Button` when it has a label.
