# Arcade homage research log

Use this when an agent is asked to build a GameMaker game by studying an
existing arcade program. The game repo holds the passes. This file is only
the method.

A playable sketch is not the homage. A smoke transcript that pops one enemy
does not prove a title, a high-score table, a demo, or a level marker.

## Where the notes go

Keep the trail in the game repository, in `analysis/`, the way a long reverse
engineering effort keeps one markdown file per pass plus a check script.

Each pass has:

1. The question, in one sentence.
2. An evidence label on every claim.
3. A check that reads the local program images and exits 0 only if the bytes
   still say what the pass says.
4. A "not claimed" list.
5. The GameMaker change that is allowed after the check passes. No other
   screen gets invented in that pass.

Evidence labels:

- **Hardware-confirmed.** Board description from an emulator driver. The
  driver is not the game.
- **ROM-confirmed.** Bytes or a named instruction sequence, effect stated in
  the agent's own words.
- **Interpretation.** A name the agent gave a byte. It stays a guess until a
  later pass checks it.

## What never enters git

Do not commit program images, archives of those images, linear disassembly
listings, or pictures traced from the original tiles. Point the check at a
path outside the repo (`BB_ROM_DIR` or the project's own documented folder).
The GameMaker art is drawn for the new game. Pose counts may come from the
research. The pixels do not.

## What the agent writes down about itself

Pass 0 is allowed to be a note with no byte check: why the previous build
stopped, which screens it never opened, and which evidence it used instead
of the program. That note is part of the review. Leaving the reasoning in
the chat is how the next session repeats the sketch.

Suggested files:

```text
analysis/README.md
analysis/PASS-00-why-the-sketch-stopped.md
analysis/PASS-01-....md
analysis/check_pass01.py
analysis/OPEN-QUESTIONS.md
docs/LLM_PROJECT_LESSONS.md
```

`OPEN-QUESTIONS.md` is the list the agent is not allowed to fake. If the
flower row, the demo, or the score decimal is on that list, the game does
not ship a stand-in and call it done.

## Order

Study one screen, check it, then implement that screen. Screenshot the
screen in the headless runner before starting the next question. A full
plan of every mechanic is not required before the first checked screen.
A finished-looking game is not allowed while the open-question list still
contains the screen the person asked to see.

## Relation to the harness

The verification harness still applies once a screen exists. The transcript
asserts the behavior the pass described (five saved scores, a demo that
takes its own steps, a flower count that matches the round). The transcript
does not replace the pass.
