# Progress

## Stream 1: reading the art

- [ ] `cargo init --lib`, lib and bin in one package
- [ ] Lint policy copied from chickadee, casting and indexing decided and written down
- [ ] `cells()` with tests on a small string and on `art/z.txt`
- [ ] Binary prints art back flat
- [ ] Checkpoint: Z and S print flat, shadows gone

## Stream 2: art into a solid

- [ ] `SurfacePoint`
- [ ] Front and back faces
- [ ] Side walls only where a neighbor is empty
- [ ] Tests: everything fits in -1 to 1, walls face empty cells
- [ ] Checkpoint: tests pass, point count checked on paper

## Stream 3: rotation and the first spin

- [ ] Matrix type with quarter-turn tests
- [ ] `Spin`: turntable and tumble
- [ ] `frame()`: rotate, cull, project, z-buffer, light, ramp
- [ ] Terminal loop
- [ ] Checkpoint: the Z spins, clip recorded

## Stream 4: making it a library

- [ ] `Renderer` with defaults
- [ ] `cli` feature
- [ ] Flags
- [ ] Docs, README gif, CI
- [ ] Checkpoint: the S spins, CI green, `v0.1.0`

## Stream 5: the S on scottyfermo.com

- [ ] Design settled
- [ ] `SpinningLogo` component
- [ ] Reduced-motion still frame
- [ ] Checkpoint: spinning in `dx serve`
