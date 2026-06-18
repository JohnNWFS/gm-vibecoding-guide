# LESSONS_LEARNED_FROM_CLAUDE_COLLABORATION.md

## Purpose

This document supplements the technical GameMaker vibe-coding guides and the existing
`LESSONS_LEARNED_FROM_LONG_TERM_CHATS.md` (which reflects ChatGPT collaboration) by
capturing patterns observed across **multiple long-running GameMaker projects developed
in collaboration with Claude** via claude.ai — not Claude Code, but the standard chat
interface.

Projects contributing to these observations include:

- A BASIC interpreter built in GameMaker Studio and deployed to the web
- A procedural narrative/combat RPG simulation engine in GML
- A modernized retro arcade homage, designed via reverse engineering of the
  original ROM before any GML was written
- Various prototypes, bug-fix sessions, and architecture reviews spanning 2024–2026

These lessons are biased toward the specific working style that emerges from using
Claude in a **chat-based, paste-the-code workflow** — no git integration, no IDE
plugin, no file system access. Everything goes through the clipboard.

---

## 1. Claude's Context Limit Is a Hard Wall, Not a Fade

ChatGPT degrades gradually under long conversations — it starts forgetting things
around the edges. Claude's context limit is a cliff: the conversation ends with a
hard cutoff. This has different operational implications.

**Plan sessions around a single coherent task.** Once you commit to a conversation,
you want to finish that task before hitting the wall. Switching goals mid-session
wastes context on things that won't be useful when you transfer out.

**Watch for the warning.** Claude will sometimes signal that the context is growing
long. Treat this as a trigger to request a handoff document *before* the conversation
ends, while the model still has full access to everything discussed.

**Request a structured handoff proactively.** At any natural pause point that feels
like roughly "60-70% done," ask Claude to write a session summary formatted for
pasting into a new chat. Waiting until the conversation dies means you lose that
ability.

---

## 2. The Concatenated File Is Your Context Window Currency

Because Claude has no persistent memory of your project between sessions, every new
conversation starts cold. The fastest way to re-establish shared understanding is to
paste a concatenated dump of all relevant GML files.

A useful format:

```
/// @object obj_editor
/// @event Create
[full GML code]

/// @object obj_editor
/// @event Step
[full GML code]

/// @script scr_run_program
[full GML code]
```

Claude is effective at holding a large concatenated file in context and reasoning about
it holistically. This is one area where Claude tends to outperform lighter models — it
can scan a large code dump and spot bugs that no single file view would reveal.

**Only include files you've touched in this project phase.** A 140-file project
doesn't need 140 files in the paste. Use the concatenated file to describe the system
under discussion, not the whole game.

---

## 3. Clipboard Corruption: Underscores Become Asterisks

When copying GML from GameMaker's IDE into a Claude chat, underscores at the
beginning of certain identifiers sometimes render as asterisks. For example,
`_enq` arrives as `*enq`, while `_CAP` may come through correctly.

This is not a Claude error. It is a GameMaker IDE clipboard artifact.

**The working rule:** When Claude returns code with lone asterisks that don't make
sense as multiplication, they are almost certainly underscores from your original
source. Mentally substitute back. You can tell Claude explicitly: *"Treat all lone
asterisks as underscores from my paste — that's a clipboard artifact from GameMaker."*

This saves significant back-and-forth on apparent syntax errors that are actually
display corruption.

---

## 4. `var` Declarations in Step Events Shadow Instance Variables

This is one of the most common bugs Claude has diagnosed across long BASIC interpreter
sessions. GameMaker's `var` keyword creates a local variable scoped to that event
execution. When you declare:

```gml
var edit_line_index = 0;
var edit_col = 0;
```

...at the top of a Step event, you are resetting these values every single frame,
even if you meant to use the persistent instance variables. The instance variables
with the same names are hidden for the rest of that event.

Claude will catch this, but it cannot catch it if you only paste the Create event.
**Always paste the full Step event when reporting bugs about state that "doesn't
persist."**

---

## 5. Persistent Global Objects Carry State Between Runs

When a `persistent` object initializes global DS structures in its Create event —
`ds_map_create()`, `ds_list_create()`, etc. — and the player hits Run again without
restarting the executable, those globals are not re-created. The Create event does
not fire again. Stale state from the previous run leaks into the next.

**Real example from building a BASIC interpreter in GameMaker:** The `COLOR` command stored the last-used color in a
global. On second run, the interpreter inherited that color instead of the default
green, because the persistent `obj_globals` never re-ran its Create event.

**Pattern to follow:** Explicit reset functions that clear and rebuild DS structures,
called on `game_restart()` or on transition to the title/editor room — not just in
Create events of persistent objects.

Claude will often produce correct Create-event initialization code and miss this
lifetime issue unless you describe the symptom ("works first run, breaks on second
run without restarting the EXE").

---

## 6. Ask Claude to Spot the Bug Before Asking It to Rewrite

Claude is better at reading existing code and identifying a precise fault than it is
at generating a clean replacement when the architecture is complex. In one BASIC
interpreter project, asking "can you redo the input command so it just works"
produced architecturally mismatched code. Asking "here's the full Step event — why
doesn't the cursor position persist?" immediately identified the `var` shadowing
issue.

**For debugging sessions:** Paste the code, describe the symptom precisely, and ask
for diagnosis before asking for replacement. Let Claude identify the cause, confirm
your understanding of it, then ask for the targeted fix.

---

## 7. Multi-LLM Triangulation Is a Real Workflow

Across a narrative RPG simulation and a BASIC interpreter project, the working pattern was:

- Use **Claude** for architecture reasoning, bug diagnosis, and reviewing large
  code dumps
- Use **ChatGPT/Codex** for iterative generation on well-specified tasks
- Bring **Codex output back to Claude** to verify it against the actual codebase

This triangulation exposed a consistent failure mode: Codex would claim to have
committed fixes that either (a) never reached the remote repository, or (b) did not
actually implement the described behavior. Claude, given the Codex summary and the
actual code, could spot the discrepancy.

**For any significant Codex/ChatGPT-generated fix:** Paste the changed files into
Claude along with the intent and ask for a second opinion before running the game.
Claude does not have access to your git history and cannot check the remote, but it
can verify whether the code *does what it claims to do*.

---

## 8. Codex Git Pushes Silently Fail

During wound-spiral debugging on a procedural RPG simulation, Codex reported
"Created PR via tool" and showed a commit hash in its UI. The commit did not exist
on the remote. Running `git log --oneline` on `origin/dev` confirmed the branch had
not advanced.

**Operational rule:** Never assume a Codex git operation succeeded. Always run
`git fetch --all && git log --oneline -5` to confirm the commit actually landed before
building on top of it. If the commit is missing, the Codex sandbox committed locally
to its own isolated environment without pushing.

This is not a GML problem, but it wasted entire sessions trying to debug "fixed" code
that had never been applied.

---

## 9. Design Documents Before Code: The ROM Reverse-Engineering Model

A retro maze homage project produced three deliverables before any GML was written:
a full game design brief, an art specification, and a structured coder kickoff prompt.
This came directly from spending a session reverse-engineering an original arcade ROM
with Claude to understand what the original code was actually doing.

The result was that the coder LLM prompt specified:

- Which GML functions to use for wall collision (`tilemap_get_at_pixel`, not
  collision objects)
- Why to use step-based movement timers instead of `move_towards_point`
- Exactly how grid coordinates map to pixel coordinates
- What to stub versus what to implement in the first pass

Every one of those decisions came from understanding the source material, not from
guessing. The resulting kickoff prompt gave a code-generating LLM a much smaller
decision space.

**The lesson:** Even if you are not reverse-engineering a ROM, spending a session
with Claude talking through architecture and design before writing code produces a
better specification document than writing code and discovering architecture problems
mid-way through. Claude can help you produce that design brief interactively.

---

## 10. GML-Specific Facts Every LLM Session Should Establish

These facts are not in most LLMs' training data at the level of reliability that
prevents mistakes. Paste them into any new session before asking for GML generation:

- `keyboard_check(key)` = held. `keyboard_check_pressed(key)` = single frame.
- Virtual key constants: `vk_left`, `vk_right`, `vk_up`, `vk_down`, `vk_enter`,
  `vk_escape`, `vk_backspace`.
- `room_width` and `room_height` are built-in globals — no need to declare them.
- Objects without sprites **must** override the Draw event, or nothing will render
  and the object will be invisible with no error.
- In `.yy` object definition JSON, `"spriteId": null` is valid when a Draw event
  is overridden.
- DS types (`ds_map`, `ds_list`, `ds_stack`) are **not** garbage collected. They
  must be explicitly destroyed with `ds_map_destroy()`, etc., or they leak.
- `draw_*` functions called outside a Draw event do nothing.
- Structs (introduced in GML 2.3) are preferred over `ds_map` for new code, but
  older projects use `ds_map` extensively. Don't mix idioms mid-project.
- `global.` prefix is required for all global variables; accessing an undeclared
  global is a runtime error in strict mode.
- The `.yyp` project manifest must reference every resource. Adding a new object
  or script file to disk without updating the `.yyp` means GameMaker will not see it.

---

## 11. Provide Event Context With Every Code Paste

Correct GML placed in the wrong event produces the wrong result with no error message.
When pasting code for Claude to review, always label which object and which event
each block belongs to. When receiving generated code, always confirm placement before
copying into GameMaker.

A common mistake: initialization code generated for a Create event pasted into a
Step event. The variable resets every frame. No error. Behavior is wrong.

The `/// @object obj_name` / `/// @event Event_Name` header convention used in the
concatenated file format (see Rule 2) doubles as documentation of placement intent.

---

## 12. Claude Will Not Invent GML Function Names — But It May Use Deprecated Ones

Unlike some models, Claude does not typically fabricate GML function names. However,
it may produce code using functions from GML versions prior to the 2.3 rework, or
functions that were renamed in GameMaker 2024.x releases. If a function Claude
provides doesn't exist, search the GameMaker documentation before assuming Claude
hallucinated it — it may have existed under a different name in an older version.

Known areas of version divergence that cause real problems:
- `ds_*` functions vs struct-based alternatives
- `sprite_get_*` vs direct struct property access in newer runtimes
- `json_encode` / `json_decode` vs `json_stringify` / `json_parse`
- `alarm[0]` syntax vs `alarm_set()` function style

When targeting a specific GameMaker runtime version, state it explicitly at the start
of the session.

---

## 13. The "Only Changed Files" Rule Applies Doubly to Claude

ChatGPT's document recommends asking for only changed files. With Claude, this is
especially important because Claude will produce complete file replacements by
default if you ask for "the fixed version." This is convenient for small files but
catastrophic for large ones where a full rewrite may silently drop features that were
in the original.

**Better prompt pattern:**
> "Here is the current `scr_resolve_combat`. Produce only the changed lines and
> describe where each change goes."

For larger surgical fixes, request a diff-style response: old block, new block,
explanation of why. This makes integration into GameMaker's IDE mechanical rather
than risky.

---

## 14. Runtime Log Output Is the Ground Truth

Across debugging sessions on a procedural RPG simulation, the most reliable path to
root cause was always the runtime log — the output from `show_debug_message()` calls
— not static code inspection. The wound spiral bug (a tank character accumulating 85
wounds in a single run) was visible in the log before any code was examined. The log
showed wound counts resetting to 1 every combat beat; the code showed why.

Claude can reason effectively from formatted debug output. If you have a bug:

1. Add targeted `show_debug_message()` calls at the suspected failure points.
2. Run the game and capture the output.
3. Paste the log into Claude along with the code.

Claude will frequently identify the failure in the log before needing to read the
code in detail.

---

## 15. State the Weird Part Upfront — Claude Will Protect It

This mirrors the ChatGPT document's "Preserve the Weird Part" lesson, but the
specific failure mode differs. Claude does not strongly drift toward conventional
implementations the way ChatGPT does. Instead, Claude may:

- Re-scope an unusual mechanic toward a more mathematically tidy version
- Add safety guards that inadvertently prevent the extreme behavior you wanted
- Simplify a deliberately recursive or self-modifying structure in the name
  of maintainability

For example: in a procedural RPG simulation, the wound accumulation system was supposed to
allow characters to enter deeply negative HP states and accumulate wounds rapidly
as a representation of narrative tragedy. Claude's early fixes added caps and guards
that "saved" characters too early. The intent (catastrophic failure as story beat)
had not been communicated.

**State your design intent in terms of player experience, not mechanics:**
> "I want the tank to be able to spiral into a death march that reads as tragic in
> the log — wounds should be able to climb into double digits as a readable narrative."

Claude will write the code to serve that intent rather than optimize it away.

---

## Final Observation

Across these projects, Claude's most consistent contribution was not code generation
but **diagnosis and structural reasoning** — reading a large paste, identifying the
exact faulty assumption, and stating it precisely so a targeted fix was possible.

The workflow that produced the best outcomes:

1. Start a session with a structured handoff document and the concatenated code
2. Describe the symptom precisely, including runtime evidence
3. Ask for diagnosis before asking for code
4. Confirm the diagnosis, then request a targeted fix (not a full rewrite)
5. Request a session summary before ending any conversation that produced significant
   architectural decisions

The concatenated file, the precise symptom, and the runtime log are the three
inputs that make Claude most effective on a GameMaker project.