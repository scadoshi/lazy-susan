# lazysusan

Set your ASCII art on it and it spins.

lazysusan reads block-character art, turns it into a solid with real depth, and spins it in your terminal, shaded by a light. It's named after the spinning tray in the middle of the table.

```
▎▎▉▉▉▉▉▉▉
 ▎▎▉▉▉▉▉▉▉
      ▎▎▉▉
     ▎▎▉▉
    ▎▎▉▉
   ▎▎▉▉▉▉▉▉▉▉
    ▎▎▉▉▉▉▉▉▉▉
```

## Built by hand, on stream

Every line here is written by hand, live on Twitch as Scadoshi and in solo sessions between streams. AI is allowed as a study partner that asks questions and gives hints, never as the one writing the code. The rules are in `context/rules/hand-coding.md`.

It follows an earlier project, [torust](https://github.com/scadoshi/torust), a hand-written take on Andy Sloane's spinning ASCII donut. lazysusan takes the same rendering ideas and points them at any art you give it.

## Status

Not started. The plan is in `context/plan.md` and progress is tracked in `context/progress.md`.

## Where it's going

1. Read block art into a grid of solid and empty cells
2. Turn each cell into a small block with front, back and side walls
3. Rotate it with matrices and draw it with a z-buffer and a light
4. Package it as a library with a thin terminal binary: `lazysusan art/z.txt`
5. Run the same library in a browser to spin the S on [scottyfermo.com](https://scottyfermo.com)

## Art

`art/z.txt` is the Z from [Zwipe](https://zwipe.net) and `art/s.txt` is the S from scottyfermo.com. Any text file works: `▉` and `█` count as solid, everything else is empty space.
