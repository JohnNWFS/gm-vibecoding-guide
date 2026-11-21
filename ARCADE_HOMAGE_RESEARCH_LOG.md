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
- **Video-confirmed.** A visible sequence, composition, relative timing, or
  repeated behavior at named timestamps in an authorized playthrough.
- **Live-observed.** Behavior the owner or agent reproduced interactively,
  including the input used and the emulator/build tested.
- **Interpretation.** A name the agent gave a byte. It stays a guess until a
  later pass checks it.

## Use playthrough video as indexed evidence

When the original game has a public playthrough—or the owner can record an
authorized playthrough—ask for the video early. Do not make the LLM scrub a
multi-hour recording manually every time it needs one scene. Add a small local
tool that uses `ffprobe`/`ffmpeg` (or equivalent) to:

1. record the source resolution, frame rate, and duration;
2. make a coarse contact-sheet index across the whole playthrough;
3. extract denser timestamped frames around the screen or behavior under
   investigation; and
4. write a manifest so later sessions can reproduce the same evidence.

Keep the video and extracted frames out of Git unless their rights explicitly
allow redistribution. Store durable notes in the game repository: source
identity, timestamps, observations, implementation conclusions, and a clear
"not claimed" list.

Use the indexed frames to test the coding plan before and after implementation:

- opening and transition order;
- camera framing and native-screen composition;
- landmark order and approximate spacing;
- enemy entrance side, density, overlap, and firing cadence;
- player/vehicle orientation and animation phases;
- HUD state and visible progression; and
- the first point where the homage diverges from the original.

Measure the source video's native gameplay resolution against the GameMaker
GUI or camera size. A coordinate map can be numerically correct yet look
completely wrong when one source pixel is treated as one output pixel instead
of preserving the original screen scale.

Video is output evidence, not recovered source. It usually cannot prove the
player's exact inputs, hidden collision types, RNG state, event-table
coordinates, object-pool rules, or why an event occurred. Corroborate those
with ROM/code analysis, emulator traces, or a documented live test. If a video
only shows that enemies arrive from both sides, label hand-placed spawn points
as provisional rather than calling them decoded.

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

Study one screen, check it, then implement that screen. Compare the resulting
screen or sequence with the indexed reference frames before starting the next
question. Screenshot the screen in the headless runner when the project's
approved test path permits it. A full plan of every mechanic is not required
before the first checked screen. A finished-looking game is not allowed while
the open-question list still contains the screen the person asked to see.

## Relation to the harness

The verification harness still applies once a screen exists. The transcript
asserts the behavior the pass described (five saved scores, a demo that
takes its own steps, a flower count that matches the round). The transcript
does not replace the pass.
