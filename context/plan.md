# Plan

Six streams, from an empty folder to a library that spins any block art in a terminal, a Ratatui app or a browser. Each one ends on something you can run. Solo sessions pick up wherever the last one stopped.

Every session goes the same way: recap what landed, read, answer the questions out loud, build, and commit at the checkpoint. Stuck gets a hint, not an answer (`rules/hand-coding.md`).

```
art ──→ cells ──→ surface points ──→ frame
       grid.rs     solid.rs          render.rs
```

The library turns art into a frame of text. The binary owns the terminal and the clock, so a web page can use the library later.

## 1. Reading the art

A library that reads art into solid and empty cells, and a binary that prints it back.

Read Cargo's [Package Layout](https://doc.rust-lang.org/cargo/guide/project-layout.html), and what clippy says about `as_conversions` and `indexing_slicing`.

- Ragged rows, padded rows, or one flat `Vec`? Pick by what Stream 2's neighbor checks will need.
- Stream 5 has no filesystem. Does the library take a `&str` or a path?
- Which of chickadee's lints do you relax, and where? Write the reason next to it.

Build: `cargo init --lib`, add `main.rs`, copy chickadee's lints, write the grid reader with tests, print it back.

Done when every file in `art/` prints flat with the shadows gone.

## 2. Art into a solid

Every solid cell becomes a small block of points that know which way they face.

Read the normals part of [Donut math](https://www.a1k0n.net/2011/07/20/donut-math.html).

- What's the widest point spacing that still leaves no holes on screen?
- Where does the 1×2 cell shape get corrected, in the solid or in the projection?
- Which sides get walls, and what happens at the `▉▉▎▉▉` gap in `cairn.txt`?

Build: `SurfacePoint`, front and back faces, walls where a neighbor is empty, and tests that everything fits in -1 to 1 and every wall faces an empty cell.

Done when the tests pass and the point count matches what you work out on paper.

## 3. The first spin

The Z spinning in the terminal.

Read "Basic 3D rotations" on Wikipedia's rotation matrix page, then find those matrices hiding in torust's formula.

- Is the donut's tumble x then z, or z then x? Prove it with one vector.
- How would a test catch a wrong sign in the y matrix?
- What does `-3.0_f32 as usize` do to a bounds check written after the cast?

Build: a small matrix type with tests, turntable and tumble spins, `frame()` from torust's loop, the `░▒▓█` ramp, then the terminal loop.

Done when `cargo run --release -- art/z.txt` spins. Clip it.

## 4. A real library

A clean API, any art in one command, and CI.

Read the [API Guidelines checklist](https://rust-lang.github.io/api-guidelines/checklist.html) and Cargo's [features](https://doc.rust-lang.org/cargo/reference/features.html) page.

- Who owns the frame buffers?
- Which time API still works once drag-to-spin shows up?
- What would count as a breaking change after `v0.1.0`?

Build: a `Renderer` with defaults, terminal crates behind a `cli` feature, a few flags, docs, a README gif, and CI.

Done when the S spins, CI is green, and `v0.1.0` is tagged.

## 5. A Ratatui widget

The spin as a widget any Ratatui app can drop in, and a clip worth sharing.

Read Ratatui's `Widget` and `StatefulWidget` docs, and how `Buffer` and `Rect` fit together.

- `Widget` or `StatefulWidget`? Where does the angle live between frames?
- `frame()` fills the whole terminal today. What changes when it has to fit any `Rect`, even a tiny one?
- Ratatui cells carry a style. Does the ramp stay characters, or does brightness become color?

Build: a `ratatui` feature, the widget, an example app with the spin next to some text, and a gif.

Done when `cargo run --example ratatui` spins inside a layout and the README shows it. Then post it and tag `#ratatui`.

## 6. The S on scottyfermo.com

lazy-susan in the browser. Decide where it sits and how big before this one.

Read how the portfolio animates things today, and MDN on `prefers-reduced-motion`.

- `requestAnimationFrame` or a timer?
- Is swapping a `<pre>` 30 times a second actually slow? Profile it before deciding.
- Which angle makes the best still frame?

Build: a `SpinningLogo` component that draws `frame()` into a `<pre>`, with a still frame for reduced motion.

Done when it spins in `dx serve` and holds still with reduced motion on.

## Later

SVG input for the Halo logo, color, and more of the Zwipe wordmarks.
