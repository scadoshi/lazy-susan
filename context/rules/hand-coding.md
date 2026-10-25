# Hand-coding rules

lazysusan exists so Scotty learns the math and the Rust by writing it. A working program written by someone else would defeat the point, so these rules hold for any AI working in this repo, on stream or off.

## Scotty writes the code

- AI never writes, edits or generates Rust in this repo: no code blocks of the thing being built, no edits to `src/` or `tests/`, no "here's the function".
- General Rust that isn't the answer is fine to show: how `matches!` works, what `chars()` returns, the shape of a `match` on a tuple.
- Docs, plans and notes in `context/` can be edited when Scotty asks.

## Hints climb one rung at a time

When something's stuck, start at the bottom, and only move up after a real try at the rung below:

1. A question that points at the problem.
2. The concept involved, or a page to read.
3. The line or function where it lives.
4. The fix, described in words.

If Scotty asks outright for the answer, go to rung 4, in words, and say that's what it was.

## Before each block, questions

Every stream in `context/plan.md` has questions to answer out loud before coding. Ask them. If one can't be answered yet, that's the thing to read about, before any code.

## Reviewing is allowed

Once Scotty has written something, AI can review it: point out bugs (as hints, climbing the same ladder), naming, missing tests, and anything against the comment or commit rules. Reviewing a finished checkpoint in full is fine.
