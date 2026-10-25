# Plan

Five streams take lazy-susan from an empty folder to a library that spins any block art, in a terminal or a browser. Each stream is about two hours and ends on something you can see. Solo sessions between streams pick up wherever the last one stopped.

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

---

## Stream 1: reading the art

**Goal:** a library that reads block art into a grid of solid and empty cells, with tests, and a binary that prints it back.

**Read**
- The Cargo book, "Package Layout": a lib and a bin in one package. doc.rust-lang.org/cargo/guide/project-layout.html
- Clippy's docs for `as_conversions`, `indexing_slicing` and `cast_possible_truncation`. clippy.rust-lang.org/

**Questions**
- `cells()` gives back ragged rows. Pad to a rectangle, keep `Vec<Vec<bool>>`, or go flat with a width? Follow each choice into the neighbor lookups in Stream 2 and the centering math before picking.
- Does the library take `&str`, a path, or `impl Read`? Stream 5 runs in a browser with no filesystem; let that decide.
- Projection is float-to-cell casts and grid writes every frame. Which of chickadee's lints do you relax for the whole crate, which with a targeted `#[allow]`, and what invariant makes each cast safe? Write the invariant where the allow goes.
- Solid is `▉` and `█` today. Hard-code it or make it a parameter, and what does that decide about who the library is for?

**Build**
1. `cargo init --lib` in this folder, then add `src/main.rs` beside `src/lib.rs`.
2. Copy chickadee's `[lints.clippy]` block into `Cargo.toml`, then make the call from the last question and write the reason down as a comment.
3. `cells(art: &str) -> Vec<Vec<bool>>` in `src/grid.rs`: only `▉` and `█` are solid. Test a two-line string first, then `art/z.txt`, with the solid cells counted by hand.
4. The binary reads a file path from its arguments and prints the grid back flat, `#` for solid and a space for empty.

**Checkpoint:** every file in `art/` prints back flat with the shadows gone. The single letters are the easy case; the wordmarks have rows of different lengths and tails that hang below the line. Commit.

**You'll own:** lib versus bin, an input API that works without a filesystem, your first lint policy.

---

## Stream 2: art into a solid

**Goal:** every solid cell becomes a small block made of surface points, each knowing which way it faces.

**Read**
- Andy Sloane's "Donut math", the part on surface normals and lighting. a1k0n.net/2011/07/20/donut-math.html

**Questions**
- How far apart should points be? Too far and the letter shows holes up close, too close and you burn time. Work out the largest spacing that still lands a point in every screen cell, from the frame size and K1, instead of guessing.
- A point cloud redraws every point every frame. The other way is filling each face as a quad per frame. Estimate both for `zervice.txt` at 160×48 and say which you'd pick if this had to run on a phone.
- Cells are 1×2 and the screen doubles x. Where does the aspect correction live: in the solid, in the projection, or split? What does the letter look like if it's applied twice, or not at all?
- `cairn.txt`'s `▉▉▎▉▉` gap gives two walls facing each other one cell apart. At what angles can you see both, and what does the z-buffer do with them?

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
- Derive the y-axis matrix from the x one, then check its signs against the right-hand rule. Why is y the axis where the sign flips, and how would you catch it in a test?
- Is the donut's tumble Rz·Rx or Rx·Rz? Prove it with one vector through both, not by matching the article.
- Normals are supposed to transform by the inverse transpose. Why can you skip that here, and what change to the model would make you need it?
- Culling on the sign of the normal's z is exact for an orthographic camera, not a perspective one. What's the right test with perspective, and how wrong is the shortcut at K2 = 5?
- With indexing denied: nested `get_mut` or a flat buffer at `y * width + x`? And what does `-3.0_f32 as usize` do to a bounds check written after the cast?

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
- `frame()` allocates two buffers per call. Who owns them: the caller, the `Renderer`, or `frame` itself? Price each one at 30 frames a second in a browser.
- Where does a frame's time actually go? Measure before touching it, then pick the cheapest real win: cull earlier, thin the points, or cache more of the rotation.
- Seconds, frame number, or the caller passes the rotation itself: which API survives adding drag-to-spin later without a breaking change?
- After `v0.1.0`, what would be a breaking change, and what should stay private so it never has to be one?

**Build**
1. `Renderer { width, height, spin, ramp, light }` with `Renderer::new(width, height)` setting sane defaults.
2. The binary's dependencies behind a `cli` feature, so the library builds with none of them.
3. Flags: `--tumble`, `--ascii` (the donut ramp), `--speed`.
4. Doc comments on everything public (see `context/rules/comment-guidelines.md`), a README gif, and CI running fmt, clippy and tests.

**Checkpoint:** `cargo run --release -- art/s.txt` spins the S, CI is green, the README gif is up. Commit, tag `v0.1.0`.

**You'll own:** API design, cargo features, CI.

---

## Stream 5: the S on scottyfermo.com

**Goal:** lazy-susan running in the browser, spinning the S on the portfolio.

Settle the design before this stream: where it sits, how big, whether it replaces the navbar S or lives in the hero. Visitors who ask for reduced motion get a still frame.

**Read**
- The Dioxus docs on async and timers, and how the portfolio already animates things.
- How the portfolio takes orangui as a git dependency.
- MDN, `prefers-reduced-motion`.

**Questions**
- `requestAnimationFrame` or a fixed timer through Dioxus: which one stops when the tab is hidden, and which one fights the browser's own frame timing?
- Swapping a whole `<pre>`'s text 30 times a second: actually expensive, or fine? Find out with the browser's profiler instead of guessing.
- What does the library add to the wasm bundle, and what would you cut first if it's too much?
- Reduced motion shows a still frame. Which angle, and why that one?

**Build:** lazy-susan as a git dependency with `default-features = false`, a `SpinningLogo` component that renders `frame()` into a `<pre>`, a small fixed size, and a still frame for reduced motion.

**Checkpoint:** the S spins in `dx serve`, and holds still with reduced motion on.

**You'll own:** Rust in the browser, driving a pure library from a UI.

---

## Later

- **SVG input:** the Halo logo from its SVG path, which brings an SVG path parser and Bézier curves.
- **Color:** truecolor, shaded by brightness.
- **More art:** every Zwipe wordmark is block art already.
