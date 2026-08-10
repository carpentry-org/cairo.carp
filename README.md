# cairo.carp

Thin FFI bindings over the [Cairo 2D graphics library](https://www.cairographics.org/) for [Carp](https://github.com/carp-lang/Carp).

The goal is to expose Cairo's C API with Carp-friendly naming (kebab-case, grouped under a single `Cairo` module), without layering a higher-level drawing model on top. Sketch-style primitives live in [anima.carp](../../archive/anima.carp).

## Installation

Cairo must be installed on the system and discoverable via `pkg-config`:

```
brew install cairo        # macOS
apt install libcairo2-dev # Debian / Ubuntu
```

Then load the library:

```clojure
(load "git@github.com:carpentry-org/cairo.carp@0.2.0")
```

(or by local path during development).

## Example

```clojure
(load "cairo.carp")

(defn main []
  (let-do [surf (Cairo.image-surface-create Cairo.format-argb32 200 200)
           cr (Cairo.create surf)]
    (Cairo.set-source-rgb cr 1.0 1.0 1.0)
    (Cairo.paint cr)
    (Cairo.set-source-rgb cr 0.0 0.0 0.0)
    (Cairo.set-line-width cr 4.0)
    (Cairo.move-to cr 20.0 20.0)
    (Cairo.line-to cr 180.0 180.0)
    (Cairo.stroke cr)
    (ignore (Cairo.surface-write-to-png surf (cstr "out.png")))
    (Cairo.destroy cr)
    (Cairo.surface-destroy surf)))
```

## Coverage

The binding currently covers the subset of Cairo that `anima.carp` needs, with some headroom:

- **Surfaces:** image / PDF / SVG creation, flush, finish, destroy, status, write-to-PNG, accessors for the raw image pixel buffer.
- **Contexts:** create, destroy, status, save / restore.
- **Transforms:** translate, rotate, scale, identity-matrix, `transform`, `set-matrix` / `get-matrix`, and user ↔ device coordinate conversion for both points and distances.
- **Matrices:** `CairoMatrix` as a first-class value — identity / translate / scale / rotate constructors, the composing `matrix-translate` / `matrix-scale` / `matrix-rotate`, `matrix-multiply`, a `Result`-returning `matrix-invert`, and point / distance transformation.
- **Paths:** new-path, new-sub-path, move-to, line-to, curve-to, the relative variants, arc, arc-negative, rectangle, close-path, current-point accessors.
- **Path introspection:** `copy-path` and `copy-path-flat` read the current path back as an array of `CairoPathElement`s (`MoveTo`, `LineTo`, `CurveTo`, `ClosePath`), and `append-path` replays such an array onto a context.
- **Painting:** stroke, fill, their `-preserve` variants, paint, paint-with-alpha, clip, clip-preserve, reset-clip, and fill / stroke / path extents.
- **State:** source RGB(A), line width / cap / join, operator, antialias, fill rule, dash pattern.
- **Patterns:** linear / radial / surface patterns, color stops, extend and filter modes, pattern matrices, `set-source` and `set-source-surface`.
- **Text:** `select-font-face`, `set-font-size`, `show-text` (Cairo's built-in toy text API), plus `text-extents` and `font-extents` for measurement.
- **Groups:** `push-group` / `push-group-with-content` redirect drawing to an offscreen surface, `pop-group` hands it back as a pattern and `pop-group-to-source` installs it as the source, and `get-group-target` reports where drawing currently goes.
- **Masks:** `mask` paints the source through a pattern's alpha, `mask-surface` through a surface's.
- **Regions:** `CairoRegion` with `CairoRectangleInt` as a first-class value — creation from one or many rectangles, `region-copy`, extents, rectangle enumeration, emptiness, point and rectangle containment, equality, translation, and the intersect / subtract / union / xor families in both their region and rectangle forms.

Enums are exposed as distinct Carp types (`CairoFormat`, `CairoLineCap`, etc.) rather than raw `Int`, so type errors catch mix-ups at compile time.

Not yet bound: scaled fonts, glyph-level text. Easy to add.

## Memory management

Cairo uses reference counting internally; this binding does not manage lifetimes for you. Every `Cairo.create` must be paired with a `Cairo.destroy`, every `*-surface-create` with a `Cairo.surface-destroy`, and every `Cairo.region-create*` with a `Cairo.region-destroy`. `Cairo.pop-group` hands back a pattern that needs a `Cairo.pattern-destroy`, where `Cairo.pop-group-to-source` disposes of it for you. See the [Cairo manual](https://www.cairographics.org/manual/) for details.

## Tests

```
carp -x test/cairo.carp
```

Builds a few surfaces, draws to them, writes PNG / SVG files to the working directory, and checks that they landed on disk. Run from the library root.
