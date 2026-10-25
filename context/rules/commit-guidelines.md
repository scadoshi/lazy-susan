## Commit Guidelines

- Concise, one-line messages (multi-line only when many changes)
- Group related files logically
- No emojis
- Use `git diff` to understand changes before committing
- **Never** include AI-agent signatures in your commits
    - Example: "Co-Authored-By: Claude <noreply@anthropic.com>"
    - Never commit with attribution metadata
- Commit at every checkpoint in `context/plan.md`, so the history shows how each one landed

## Checks to run before pushing

CI arrives in Stream 4 (`.github/workflows/ci.yml`). Until then, run these by hand.

### 1. Format with nightly

`rustfmt.toml` enables four unstable options: `imports_granularity = "Module"` (one `use` line per module), `group_imports = "StdExternalCrate"` (std, then external crates, then this crate, as separate blocks), `hex_literal_case = "Lower"`, and `format_macro_matchers = true`. Stable `cargo fmt` silently skips all of them.

```bash
cargo +nightly fmt
```

### 2. Clippy, warnings are errors

```bash
cargo clippy --all-targets -- -D warnings
```

Lint levels live in `Cargo.toml` `[lints.clippy]` (added in Stream 1, copied from chickadee). The panic family (`unwrap`, `expect`, `panic`, indexing, slicing) is denied outside tests; `clippy.toml` allows it inside them.

### 3. Tests

```bash
cargo test
```
