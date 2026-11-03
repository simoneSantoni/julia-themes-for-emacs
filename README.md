# julia-themes-for-emacs

Light and dark Emacs color themes built from the four colors of the [Julia logo](https://github.com/JuliaLang/julia-logo-graphics): blue, green, red and purple. They use the typography and wide package coverage of Emacs's bundled **Leuven** theme.

The themes do not copy Leuven's face definitions. When a theme loads, it reads Leuven's face specs and recolors each one onto the Julia palette. Every face Leuven styles, about 790 across Org, Gnus, Magit, diff, and more, therefore gets a Julia color. Leuven's font families, heights, weights and inheritance are kept. A short list of explicit faces then pins the meaning of syntax, status, Org and mode-line colors.

## The themes

| Theme | Paper | Surface (code blocks, inactive regions) |
|---|---|---|
| `julia-leuven-light` | white `#ffffff` | cool gray `#f1f3f8` |
| `julia-leuven-light-green` | pale green `#f1f8f0` (Julia green mixed with 93% white) | green wash `#e7f2e5` |
| `julia-leuven-dark` | charcoal `#1e2028` | `#292d39` |

## Palette

| Role | Logo color | Light themes | Dark theme |
|---|---|---|---|
| blue | `#4063D8` | `#4063d8` | `#96adff` |
| green | `#389826` | `#287522` (darkened for legibility on white) | `#8bd17c` |
| red | `#CB3C33` | `#cb3c33` | `#ff968c` |
| purple | `#9558B2` | `#9558b2` | `#ce9de5` |
| text | | `#282936` | `#e3e4ec` |
| muted text | | `#656979` | `#a1a5b6` |

Background tints, such as heading bands, the region and highlighted TODO keywords, all come from these four hues. Each tint is 14% accent and 86% paper.

## Color roles

The roles start from Leuven's own mapping, with blue and red swapped and then red and green swapped. As a result:

| Faces | Color |
|---|---|
| keywords, function names | green |
| strings, docstrings | red |
| constants | blue |
| types, variables | purple |
| comments | muted, italic |
| `success` / `warning` / `error` | red / purple / blue (bold) |
| cursor, minibuffer prompt, isearch | purple |
| region | green tint |
| links (`link`, `org-link`) | blue, underlined |

### Org

All four accents show up in an ordinary Org document, even one without four levels of headings:

- **Headings** cycle green, red, blue, purple through levels 1–8. Level 1 is drawn 1.3× larger. Levels 1–3 sit on a tinted band that runs to the window edge, and levels 1–2 also get an overline. The same faces apply to `outline-N`.
- **Document title**: purple, bold, 1.8× height. The `#+title:` keyword gets a purple tint.
- **Drawers, properties, block delimiters and `~code~`** are red. Blocks sit on the surface color, in `fixed-pitch`.
- **TODO** is red on a red tint, and **DONE** is green on a green tint. Tables and agenda dates are blue. Tags, dates, `=verbatim=` and the agenda structure are purple. Today's agenda date is green.
- Custom TODO keyword faces are colored if your configuration defines them: `os-org-todo-next` green, `os-org-todo-wait` blue, `os-org-todo-hold` purple, `os-org-todo-cancelled` muted. Without those definitions they do nothing.

### Mode line

- The active mode line is purple-tinted with dark text in the light themes, and solid Julia purple `#9558b2` with light text in the dark theme. Both have a purple border.
- The inactive mode line is light gray with dark text in all three themes.
- [doom-modeline](https://github.com/seagle0128/doom-modeline) is supported: its bar matches the purple or gray border, and in the dark theme its foreground is left unspecified, so active and inactive windows each pick the right text color.

### Terminals

`term-color-*` and `vterm-color-*` (16 colors, with foreground and background both set because vterm queries each one separately) use the same channel swaps as the syntax roles:

| ANSI slot | Julia color |
|---|---|
| red | blue |
| green | red |
| yellow, magenta | purple |
| blue, cyan | green |

Programs that print standard ANSI colors therefore match the buffer colors. For Julia ≥ 1.13 this includes the native REPL highlighting, using a `StyledStrings` face map that follows the same swaps (see [Julia REPL](#julia-repl)).

## How the recoloring works

`julia-leuven--define` in `julia-leuven-common.el` walks every face spec in Leuven's `theme-settings` and replaces each color value:

- **Near-neutral colors** (HSV saturation < 0.12) become paper or surface when used as a background, depending on whether they were almost white. Used as a foreground, they become text or muted, depending on whether they were dark.
- **Saturated colors** go to an accent by hue: reds to blue, oranges and yellows to purple, yellow-greens and greens to red, cyans and blues to green, violets and magentas to purple. Backgrounds get the tinted version of the accent.

The explicit faces listed above are then layered on top. All specs are collected into one alist before a single `custom-theme-set-faces` call. Calling it repeatedly while loading a disabled theme can append duplicate settings, and then Leuven's inherited spec can override the explicit one.

Leuven is loaded with `NO-ENABLE` (`(load-theme 'leuven t t)`), so only its settings are read. It is never applied. Disabling a Julia theme therefore restores the faces cleanly.

The themes never install or force a font. Leuven's Sans Serif variations, such as mail headers and scaled titles, still work, and so does whatever `fixed-pitch` font you use. JuliaMono suits these themes well.

## Requirements

- GNU Emacs with the bundled `leuven` theme, which ships in `etc/themes/`. Tested on Emacs 30.2.
- Only built-in libraries: `color` and `cl-lib`.

## Installation

Clone the repository and add it to `custom-theme-load-path`:

```sh
git clone https://github.com/simoneSantoni/julia-themes-for-emacs.git ~/src/julia-themes-for-emacs
```

```elisp
(add-to-list 'custom-theme-load-path "~/src/julia-themes-for-emacs")
(load-theme 'julia-leuven-light t)   ; or julia-leuven-light-green, julia-leuven-dark
```

Each `*-theme.el` file loads `julia-leuven-common.el` from its own directory, so keep the four files together.

Emacs themes stack instead of replacing each other. Disable the current theme before switching, or the old faces show through wherever the new theme says nothing:

```elisp
(mapc #'disable-theme custom-enabled-themes)
(load-theme 'julia-leuven-dark t)
```

If you load the theme from your init file, do not also keep `custom-enabled-themes` in the Custom block at the end of that file. That block runs later, and its setter disables every other theme.

### Switching between the three themes

```elisp
(defun julia-leuven-select (theme)
  "Disable the current themes and enable the Julia Leuven THEME."
  (interactive
   (list (intern (completing-read
                  "Julia theme: "
                  '("julia-leuven-light" "julia-leuven-light-green"
                    "julia-leuven-dark")
                  nil t))))
  (mapc #'disable-theme custom-enabled-themes)
  (load-theme theme t))

(defun julia-leuven-cycle ()
  "Cycle Julia Leuven themes: white, pale green, dark."
  (interactive)
  (let* ((themes '(julia-leuven-light julia-leuven-light-green julia-leuven-dark))
         (current (seq-find (lambda (th) (memq th custom-enabled-themes)) themes)))
    (julia-leuven-select (or (cadr (memq current themes)) (car themes)))))
```

## Julia REPL

Julia 1.13 and later highlight REPL input natively. To make that highlighting follow the same channel swaps, so it matches the theme in `vterm`, `julia-repl` or any terminal that uses these colors, load this in the REPL's startup code. ANSI names are used instead of hex colors, so switching between the light and dark themes still picks the right shades:

```julia
if VERSION >= v"1.13"
    import JuliaSyntaxHighlighting, StyledStrings
    for (name, colour) in (
        :julia_keyword => :blue,  :julia_funcall => :blue,
        :julia_funcdef => :blue,  :julia_builtin => :blue,      # ANSI blue  -> green
        :julia_type => :magenta,  :julia_macro => :magenta,
        :julia_symbol => :magenta,                              # -> purple
        :julia_string => :green,  :julia_string_delim => :green,
        :julia_char => :green,    :julia_char_delim => :green,
        :julia_cmd => :green,     :julia_regex => :green,       # ANSI green -> red
        :julia_number => :red,    :julia_bool => :red,
        :julia_singleton_identifier => :red,                    # ANSI red   -> blue
        :julia_comment => :bright_black,
    )
        StyledStrings.loadface!(name => StyledStrings.Face(foreground = colour))
    end
end
```

## Files

| File | Contents |
|---|---|
| `julia-leuven-common.el` | Palette, the Leuven recoloring, and the explicit face overrides (`julia-leuven--define`) |
| `julia-leuven-light-theme.el` | `julia-leuven-light`: white paper |
| `julia-leuven-light-green-theme.el` | `julia-leuven-light-green`: pale green paper |
| `julia-leuven-dark-theme.el` | `julia-leuven-dark`: charcoal |
