# LESSONS_LEARNED_FROM_LONG_TERM_CHATGPT_COLLABORATION.md

## Purpose

This document supplements the technical GameMaker vibe-coding guides by documenting observations gathered from **years of collaborative GameMaker development using ChatGPT** across numerous unrelated projects.

Unlike the other documents in this repository, this file is **not** intended to teach GameMaker or GML syntax. Instead, it captures behavioral patterns that consistently produced better results while collaborating with ChatGPT on GameMaker projects.

These lessons were learned through repeated experimentation, debugging, architecture discussions, reverse engineering, interpreter development, procedural generation, UI development, simulation projects, and game prototypes.

Many of these recommendations apply equally well to other LLMs, but they originated from long-term collaboration with ChatGPT rather than Codex.

---

# 1. Build The Smallest Playable Loop First

The fastest path to a successful project is almost never implementing every planned system.

Instead:

* Identify the core player activity.
* Build the smallest complete gameplay loop.
* Make it playable.
* Expand outward only after that loop is enjoyable.

Think vertically rather than horizontally.

Good:

```
Input
↓

Simulation
↓

Result
↓

Decision
↓

Repeat
```

Poor:

* Inventory
* Combat
* Economy
* Save System
* Dialogue
* Quest System
* UI
* AI

...all partially implemented simultaneously.

---

# 2. Prototype Systems Before Art

Early development should prioritize systems over appearance.

Placeholder graphics are perfectly acceptable:

* rectangles
* circles
* text labels
* colored boxes
* debug overlays

Artwork should replace placeholders only after mechanics are functioning correctly.

---

# 3. Request Architecture Before Code

Before asking ChatGPT to generate code, ask it to describe:

* object structure
* scripts
* ownership
* data flow
* initialization order
* event responsibilities
* persistent state

This prevents large architectural mistakes before they occur.

---

# 4. Separate Simulation From Rendering

Draw events should display information.

They should not perform game logic.

Preferred architecture:

```
Simulation
    ↓
Game State
    ↓
Rendering
```

Rendering should never become the source of truth.

---

# 5. Give Every Concept One Owner

Every important piece of state should have exactly one authoritative owner.

Bad:

* obj_game owns score
* obj_player owns score
* obj_ui owns score

Good:

* one object owns score
* every other object reads it

Duplicate ownership creates synchronization bugs.

---

# 6. Prefer Expanding Existing Systems

ChatGPT often solves problems by creating new scripts and new objects.

Before adding new resources, ask:

> Can an existing system own this behavior?

Keeping systems centralized improves maintainability.

---

# 7. Build Debug Tools Early

Developer tools save enormous amounts of time.

Useful debug displays include:

* current state
* FPS
* selected object
* room
* collision state
* AI state
* inventory
* mission status
* player variables

Leave these enabled until late development.

---

# 8. Build Command Consoles Before Fancy Menus

Many prototypes become dramatically easier to test with a simple command interface.

Examples:

```
HELP
RESET
SAVE
LOAD
NEXT
SPAWN
DEBUG
```

Even if removed later, command consoles accelerate development.

---

# 9. Separate Data From Behavior

Game data should exist independently from gameplay logic.

Game logic should exist independently from rendering.

Keep these layers separate whenever possible.

---

# 10. Seed Representative Test Data

Avoid dozens of identical placeholder entries.

Instead create a handful of varied examples.

Good:

* weak enemy
* strong enemy
* ranged enemy
* armored enemy
* cursed enemy

Representative data exposes balancing issues earlier.

---

# 11. Verify Runtime Behavior

Source code inspection is not enough.

Whenever possible:

* run the game
* examine screenshots
* review recordings
* compare runtime behavior against expectations

Reality frequently disproves assumptions.

---

# 12. Separate Facts From Hypotheses

When reverse engineering or debugging:

Maintain separate categories:

* Confirmed
* Likely
* Speculative
* Unknown

Do not silently promote assumptions into facts.

---

# 13. Request Only Changed Files

As projects grow larger, ask ChatGPT to return:

> Only files that changed.

This reduces accidental regressions and conversation bloat.

---

# 14. Keep State Machines Explicit

Avoid arbitrary string comparisons throughout the project.

Prefer enums or constants.

Example:

```
MODE_MENU
MODE_PLAY
MODE_PAUSE
MODE_SHOP
MODE_DIALOG
MODE_RESULTS
```

Maintain a single authoritative state variable.

---

# 15. Centralize Initialization

Initialization scattered across many Create events becomes difficult to maintain.

Whenever practical:

* initialize globals together
* initialize sample data together
* initialize systems together

Startup should be understandable from one location.

---

# 16. Build Vertically

Build complete slices rather than partial systems.

Preferred order:

```
Architecture
↓

Minimal Gameplay Loop
↓

Verification
↓

Expansion
↓

Refactoring
↓

Repeat
```

This consistently produces better results than broad feature-first development.

---

# 17. Prototype Graphics With Primitive Shapes

Before importing assets:

* rectangles
* circles
* outlines
* labels
* placeholders

Gameplay quality should not depend on final artwork.

---

# 18. Build Developer Utilities

Developer-only functionality pays for itself quickly.

Useful examples:

* room jump
* object spawn
* variable editor
* inventory injection
* mission simulator
* state viewer
* reset game
* quick save/load

These dramatically accelerate testing.

---

# 19. Request Placement Instructions

Correct GameMaker code placed into the wrong event is still incorrect.

Whenever requesting code, ask ChatGPT to specify:

* object
* event
* script
* creation instructions

Placement matters.

---

# 20. Keep UI Layout Stable

Early prototypes benefit from persistent screen regions.

Typical layout:

* Header
* Main viewport
* Status panel
* Log window
* Console
* Buttons

Stable layouts simplify iteration.

---

# 21. Use Shared Backend Actions

Buttons, keyboard shortcuts, and command consoles should all invoke the same backend functions.

Avoid duplicated logic between input methods.

---

# 22. Separate Engine From Content

The engine should not know specific missions, enemies, levels, or cards.

Content should populate engine systems.

This improves flexibility and maintainability.

---

# 23. Avoid Giant Step Events

Very large Step events become difficult for both humans and LLMs to reason about.

Prefer:

* helper scripts
* focused functions
* modular responsibilities

Large monolithic events should be refactored.

---

# 24. ChatGPT Will Drift Toward Conventional Designs

When presented with unusual ideas, ChatGPT often attempts to normalize them into familiar mechanics.

Periodically restate:

* player fantasy
* design pillars
* unique mechanics
* project goals

This keeps implementation aligned with the original vision.

---

# 25. Preserve The Weird Part

The unusual mechanic is often the identity of the game.

Do not simplify it into something conventional merely because implementation becomes easier.

Protect the original concept throughout development.

---

# 26. Build One Layer At A Time

The highest quality collaborative development tends to follow:

```
Architecture
↓

Data Structures
↓

Simulation
↓

User Interface
↓

Content
↓

Polish
↓

Optimization
```

Mixing phases usually reduces quality.

---

# 27. Maintain A Single Source Of Truth

Each important value should exist in one canonical location.

Simulation updates it.

UI displays it.

Save systems serialize it.

Everything else references it.

Duplicate copies create bugs.

---

# 28. Test Every Completed Subsystem

Do not wait until the game is "finished."

Each completed subsystem should immediately be:

* run
* tested
* stressed
* verified

Frequent testing catches architectural mistakes early.

---

# 29. Runtime Evidence Overrides Reasoning

When runtime behavior contradicts architectural assumptions, trust runtime evidence.

Screenshots, debugger output, recordings, and live execution have higher confidence than speculation.

For GameMaker projects with an autotest harness, use this **priority order**:

1. Machine-readable transcript with `TEST: ... = PASS|FAIL`
2. Screenshot or capture at a documented path
3. Targeted debug messages for one hypothesis
4. Igor compile log (build failures)
5. Source-code reasoning alone

Igor exit code is unreliable when the Runner blocks on "press key to exit" after a successful run. See `GM_CLI_HEADLESS_VERIFY.md` in this repository.

---

# 31. Build A Transcript, Not A Console Dump

Agents drown in `show_debug_message` output mixed with compiler spam.

Projects that write a **short transcript file** per run (header, assert lines, footer) let agents verify in seconds. The pattern is documented in `LLM_VERIFICATION_HARNESS_PATTERN.md`. Adopt it early on interpreter-like or test-heavy GameMaker projects.

---

# 30. Preserve The Player Fantasy

Perhaps the single most valuable lesson learned from long-term collaboration:

ChatGPT naturally optimizes toward conventional implementations.

Regularly remind the model:

* What fantasy is the player buying?
* What makes this game different?
* What should never be optimized away?

Protect the experience first.

Implementation exists to serve the fantasy.

---

# Final Observation

Across years of collaborative GameMaker development with ChatGPT, the greatest productivity gains did **not** come from generating more code.

They came from:

* reducing ambiguity
* clarifying architecture
* separating responsibilities
* validating assumptions through runtime testing
* maintaining a single source of truth
* building complete gameplay loops before expanding systems

Good architecture consistently outperforms clever code.
