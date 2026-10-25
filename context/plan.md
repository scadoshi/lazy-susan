# Plan

Five streams take lazysusan from an empty folder to a library that spins any block art, in a terminal or a browser. Each stream is about two hours and ends on something you can see. Solo sessions between streams pick up wherever the last one stopped.

## How a session runs

1. **Recap, 5 minutes.** What landed last time, what's next, the checkpoint for today.
2. **Read, 20 minutes.** The pages listed for the stream, out loud.
3. **Questions.** Answer the stream's questions out loud before writing code. A question you can't answer is the next thing to read about.
4. **Build.** Tests first where the plan says so. Stuck means a hint, not an answer (`context/rules/hand-coding.md`).
5. **Checkpoint.** Run it, look at it, commit it.
6. **Wrap, 5 minutes.** What was hard, what's next.

If a stream runs short, start the next one. If it runs long, stop at the last green test and pick up there. The plan doesn't care how many nights it takes.

## The shape of the thing

```
art (text) ──→ cells ──→ surface points ──→ frame (text)
              grid.rs     solid.rs          render.rs
                                               ↑
                                     spin, light, ramp, size

src/lib.rs    the library: no terminal, no clock, no sleeping
src/main.rs   the terminal binary: reads a file, loops, prints
```

The library takes art in and gives a frame of text back. Everything about time and the terminal lives in the binary, so a web page can drive the same library later.

## Words you'll use

- **Cell:** one character of the art. About twice as tall as it is wide.
- **Surface point:** a point on the solid's skin, plus its normal.
- **Normal:** the direction a surface faces, length 1. Lighting is the normal against the light.
- **Projection:** 3D to 2D. Divide by depth so far things shrink: x' = K1·x / (K2 + z).
- **ooz:** one over z. Bigger means closer. Stored in the z-buffer.
- **Z-buffer:** per screen cell, the closest ooz so far. A point only draws if it's closer.
- **Ramp:** characters from dark to bright. `░▒▓█` here, `.,-~:;=!*#$@` for the donut.
- **Turntable and tumble:** turning around the vertical axis (readable) versus around two axes at once (the donut).

---

## Stream 1: reading the art

**Goal:** a library that reads block art into a grid of solid and empty cells, with tests, and a binary that prints it back.

**Read**
- The Cargo book, "Package Layout": a lib and a bin in one package. doc.rust-lang.org/cargo/guide/project-layout.html
- The Rust book, "Storing UTF-8 Encoded Text with Strings", the part about bytes versus chars. doc.rust-lang.org/book/ch08-02-strings.html
- Clippy's docs for `as_conversions`, `indexing_slicing` and `cast_possible_truncation`. clippy.rust-lang.org/

**Questions**
- `▉` looks like one character. How many bytes is it? What does `line.len()` give you versus `line.chars().count()`?
- Each row of `art/z.txt` starts with `▎▎`. Is that part of the letter? What does it turn into once the letter is a real 3D solid?
- A terminal cell is about twice as tall as wide. If a cell is 1 wide, how tall is it?
- What should the library know about the terminal? What should the binary know about the math?
- chickadee denies `indexing_slicing` and warns on `as` casts, and CI treats warnings as errors. Projection is all float-to-cell casts and grid lookups. Keep the rules and work around them, or allow some of them on purpose? What's the cost either way?

**Build**
1. `cargo init --lib` in this folder, then add `src/main.rs` beside `src/lib.rs`.
2. Copy chickadee's `[lints.clippy]` block into `Cargo.toml`, then make the call from the last question and write the reason down as a comment.
3. `cells(art: &str) -> Vec<Vec<bool>>` in `src/grid.rs`: only `▉` and `█` are solid. Test a two-line string first, then `art/z.txt`, with the solid cells counted by hand.
4. The binary reads a file path from its arguments and prints the grid back flat, `#` for solid and a space for empty.

**Checkpoint:** `cargo run -- art/z.txt` and `cargo run -- art/s.txt` print both letters flat with the shadows gone. Commit.

**You'll own:** lib versus bin, chars versus bytes, `include_str!` versus reading a file, your first lint policy.

---

## Stream 2: art into a solid

**Goal:** every solid cell becomes a small block made of surface points, each knowing which way it faces.

**Read**
- Andy Sloane's "Donut math", the part on surface normals and lighting. a1k0n.net/2011/07/20/donut-math.html
- Then picture a sugar cube on the table. Name the direction each face points.

**Questions**
- The art is `cols` wide and `rows × 2` tall. How do you move its middle to (0, 0) and fit the larger side into -1 to 1? Which way does y go, and which way do you want it to go?
- Two solid cells sit side by side. Is there a wall between them? How does the grid tell you?
- What's the normal of the front face? The back? A left wall? A top wall?
- If face points sit exactly on cell edges, neighbors double up. Where should they sit instead?
- The edge of the grid counts as what: solid or empty?

**Build**
1. `SurfacePoint { point: [f32; 3], normal: [f32; 3] }`.
2. Faces in `src/solid.rs`: a few points per cell at z = -T and z = +T, with normals pointing out of each face.
3. Walls: for each side of a solid cell whose neighbor is empty or off the grid, a strip of points from back to front, facing that neighbor.
4. Tests: every point fits in -1 to 1, and every wall faces an empty cell (step a little along the normal and look up the cell you land in).

**Checkpoint:** the tests pass and the point count makes sense when you work it out on paper. Commit.

**You'll own:** normals, centering and scaling, neighbor checks, testing geometry without seeing it.

---

## Stream 3: rotation and the first spin

**Goal:** the Z spinning in the terminal.

**Read**
- Wikipedia, "Rotation matrix", only "Basic 3D rotations".
- Look back at the donut's long rotation formula in torust and find the matrices hiding in it.

**Questions**
- Write the x-axis rotation matrix from memory. Where do sin and cos go?
- Rotate around x then z, or z then x. Same result? When you multiply two matrices, which one applies first?
- A letter tumbling like the donut is unreadable half the time. Which one axis keeps it readable? What does a small fixed tilt add?
- Why do normals get the same rotation as the points?
- A point whose normal faces away from you after rotating: draw it or skip it? Which number tells you?
- With indexing denied, how do you write to `buffer[y][x]`? And does that remove the need for a bounds check before the cast? (What does `-3.0_f32 as usize` give you?)

**Build**
1. A small matrix type in `src/render.rs`: rotations around x, y and z, `multiply`, `apply`. Test quarter turns: rotating y around x a quarter turn gives z.
2. A `Spin` enum with `Turntable { tilt }` and `Tumble`.
3. `frame(points, t) -> String`, built from torust's render loop: rotate, skip back faces, project, bounds-check, z-buffer, light, ramp.
4. Ramp `░▒▓█`, plus a little ambient light so faces turned away from the light don't vanish.
5. The binary loops: clear once, then `\x1b[H`, the frame, flush, sleep about 30 ms.

**Checkpoint:** `cargo run --release -- art/z.txt` spins the Z. Record a clip. Commit.

**You'll own:** rotation matrices and their order, back-face culling, the render loop rebuilt from memory instead of from the article.

---

## Stream 4: making it a library

**Goal:** a clean public API, anyone's art in one command, and CI.

**Read**
- The Rust API Guidelines checklist, naming and documentation. rust-lang.github.io/api-guidelines/checklist.html
- The Cargo book, "Features", the part on optional dependencies. doc.rust-lang.org/cargo/reference/features.html
- chickadee's `.github/workflows/ci.yml`.

**Questions**
- What does a caller have to pass, and what should have a default?
- The terminal-size crate means nothing in a browser. How does the library avoid depending on it?
- Should `frame` take time in seconds or a frame number? Which is easier for a web page that redraws whenever it gets the chance?
- What in the public API would be painful to change later?

**Build**
1. `Renderer { width, height, spin, ramp, light }` with `Renderer::new(width, height)` setting sane defaults.
2. The binary's dependencies behind a `cli` feature, so the library builds with none of them.
3. Flags: `--tumble`, `--ascii` (the donut ramp), `--speed`.
4. Doc comments on everything public (see `context/rules/comment-guidelines.md`), a README gif, and CI running fmt, clippy and tests.

**Checkpoint:** `cargo run --release -- art/s.txt` spins the S, CI is green, the README gif is up. Commit, tag `v0.1.0`.

**You'll own:** API design, cargo features, CI.

---

## Stream 5: the S on scottyfermo.com

**Goal:** lazysusan running in the browser, spinning the S on the portfolio.

Settle the design before this stream: where it sits, how big, whether it replaces the navbar S or lives in the hero. Visitors who ask for reduced motion get a still frame.

**Read**
- The Dioxus docs on async and timers, and how the portfolio already animates things.
- How the portfolio takes orangui as a git dependency.
- MDN, `prefers-reduced-motion`.

**Questions**
- How does a component redraw about 30 times a second without a `sleep` loop?
- How many points can a phone afford per frame? How do you thin them out?
- How does the page know the visitor asked for reduced motion?

**Build:** lazysusan as a git dependency with `default-features = false`, a `SpinningLogo` component that renders `frame()` into a `<pre>`, a small fixed size, and a still frame for reduced motion.

**Checkpoint:** the S spins in `dx serve`, and holds still with reduced motion on.

**You'll own:** Rust in the browser, driving a pure library from a UI.

---

## Later

- **SVG input:** the Halo logo from its SVG path, which brings an SVG path parser and Bézier curves.
- **Color:** truecolor, shaded by brightness.
- **More art:** every Zwipe wordmark is block art already.
