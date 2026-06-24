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

- **Surfaces:** image / PDF / SVG creation, flush, finish, destroy, write-to-PNG, accessors for the raw image pixel buffer.
- **Contexts:** create, destroy, status, save / restore.
- **Transforms:** translate, rotate, scale, identity-matrix.
- **Paths:** new-path, new-sub-path, move-to, line-to, curve-to, the relative variants, arc, arc-negative, rectangle, close-path.
- **Painting:** stroke, fill, their `-preserve` variants, paint, paint-with-alpha, clip, clip-preserve, reset-clip.
- **State:** source RGB(A), line width / cap / join, operator, antialias, fill rule.
- **Text:** `select-font-face`, `set-font-size`, `show-text` (Cairo's built-in toy text API).

Enums are exposed as distinct Carp types (`CairoFormat`, `CairoLineCap`, etc.) rather than raw `Int`, so type errors catch mix-ups at compile time.

Not yet bound: patterns, matrices as first-class values, `set-dash`, `text-extents`, font extents, scaled fonts, Cairo's lower-level path introspection. Easy to add.

## Memory management

Cairo uses reference counting internally; this binding does not manage lifetimes for you. Every `Cairo.create` must be paired with a `Cairo.destroy`, and every `*-surface-create` with a `Cairo.surface-destroy`. See the [Cairo manual](https://www.cairographics.org/manual/) for details.

## Tests

```
carp -x test/cairo.carp
```

Builds a few surfaces, draws to them, writes PNG / SVG files to the working directory, and checks that they landed on disk. Run from the library root.
