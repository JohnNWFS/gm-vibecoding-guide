# Best Practices For LLM Vibecoding For GML

This guide is for Codex, Claude Code, ChatGPT, and other LLM agents working on
GameMaker Studio / GML projects. It supplements official GameMaker
documentation with field-tested practices that prevent common agent failures.

The purpose is simple: move carefully, prove runtime state, and avoid the
GameMaker-specific traps that make an agent chase symptoms for an hour.

## How To Use This File

Before changing a GameMaker project:

1. Read this file.
2. Read `GM_CLI_HEADLESS_VERIFY.md` and `LLM_VERIFICATION_HARNESS_PATTERN.md`
   when the project supports shell runs or file-based verification.
3. Read the project's own docs, especially any LLM brief, status file, or
   `docs/AUTOTEST_WORKFLOW.md`.
4. Inspect the relevant `.yy` metadata as well as the `.gml` code.
5. Form one testable hypothesis.
6. Add the smallest safe change.
7. Verify in the actual target runtime — prefer transcript/snapshot files over
   raw Igor stdout.

If the project has a local best-practices or lessons-learned file, update that
first. Only update this shared guide when the lesson is broadly useful across
GameMaker projects.

## Prime Directive

Do not guess your way through GameMaker bugs.

For a visible bug, first prove:

- current room
- active instance counts
- persistent objects in play
- relevant global state
- which event is actually running
- which build target is actually being tested

Then patch.

## GameMaker Metadata Is Code

Agents often edit `.gml` and forget that `.yy` metadata controls whether the
code exists in the runtime.

Check these every time:

- object `eventList`
- room layer `instances`
- room `instanceCreationOrder`
- project `.yyp` resource entries
- resource order files when the project uses them
- object `persistent` flags
- room dimensions and view settings
- extension `copyToTargets`

### New Script Checklist

Adding `scripts/my_feature/my_feature.gml` is not enough. Agents often omit
metadata and the function never links.

Verify all of:

- `scripts/my_feature/my_feature.gml`
- `scripts/my_feature/my_feature.yy`
- parent folder entry in `Project.yyp`
- `resource_order` entry when the project uses `*.resource_order`

Symptoms when missing: compile succeeds but `undefined` at runtime, command
never dispatches, or IDE shows the script only after manual refresh.

## Legacy GMK Recovery

Old pre-Studio projects may not import cleanly into modern GameMaker, and a
failed import is not the end of the job.

When recovering `.gmk`, `.gm81`, `.gmd`, `.gb1`, or similar projects:

- inventory every sibling copy, backup, numbered version, and exported build
- compare file sizes and dates before choosing a source
- preserve originals and work from copies
- try official/legacy importers, but expect very old files to fail
- attempt read-only asset and string extraction when import fails
- separate confirmed extracted facts from design inference
- rebuild in modern LTS incrementally, with git checkpoints
- keep a recovery report of sources inspected, tools attempted, recovered
  assets/code, inferred design, deviations, and validation commands

If the old project cannot be imported, a useful modernization can still be built
from extracted assets, object/script names, screenshots, exported builds, and
careful behavioral reconstruction.

### Room Instances Need Two Entries

For an object placed in a room, the instance usually must appear in both:

- the room layer's `instances` array
- the room's `instanceCreationOrder` array

If it appears only in the visual layer, GameMaker may show it in the editor or
serialize it, but the runtime may not create it as expected. This is a common
cause of "the object is in the room but does nothing."

## Event Wiring

GameMaker event files and event metadata must agree.

Examples:

- ordinary Draw event: `eventType: 8`, `eventNum: 0`, often `Draw_0.gml`
- Draw GUI event: `eventType: 8`, `eventNum: 64`, often `Draw_64.gml`
- Step event: `eventType: 3`, `eventNum: 0`, often `Step_0.gml`
- Create event: `eventType: 0`, `eventNum: 0`, often `Create_0.gml`

Do not assume a file named `DrawGUI_0.gml` will be wired automatically. The
object `.yy` event metadata decides what GameMaker compiles.

If a Draw GUI event is missing at runtime, inspect the object's `.yy` file
before rewriting rendering code.

## Avoid Built-In Name Collisions

GameMaker has many built-in variables and function names. Some are writable
instance fields, some are read-only functions, and some names compile in one
context but fail in another.

Do not casually introduce locals or instance variables named like common
GameMaker built-ins, including:

- `score`
- `health`
- `speed`
- `direction`
- `image_index`
- `image_speed`
- `object_index`
- `id`
- `x`
- `y`
- `room`
- `path`

Prefer specific names such as `pred_pressure`, `move_speed`, `target_id`,
`candidate_score`, or `resource_path`.

Before compiling a large LLM edit, run a quick scan for risky declarations. For
example, in PowerShell:

```powershell
rg -n "\bvar\s+(score|health|speed|direction|target|image_index|image_speed|object_index|id|x|y|room|path)\b" objects -g '*.gml'
```

Treat hits as review prompts, not automatic errors; some projects may
intentionally use particular names.

## Keyboard Constants And `ord()`

Do not use `ord()` for punctuation, symbols, or special keys with
`keyboard_check*()` functions. GameMaker's keyboard-check functions only detect
`ord()` values reliably for one-character strings in `0`-`9` or uppercase
Roman `A`-`Z`.

Use `vk_*` constants for function keys, arrows, modifiers, numpad keys, and
other special keys. For example, prefer `vk_f10` for a forensic/debug toggle
over trying to detect tilde or backtick with `ord()`.

## Persistent Objects

Persistent objects are powerful and dangerous.

Before making an object persistent, answer:

- Should this object keep running in every room?
- Should its Step event process input outside its home room?
- Should its Draw event draw outside its home room?
- Can there be duplicate instances after returning to a room?

Common safe pattern:

```gml
/// Step event for editor-only objects
if (room != rm_editor) exit;
```

Common Draw pattern:

```gml
/// Draw event for editor-only objects
if (room != rm_editor) exit;
```

If a persistent editor object keeps accepting Enter while an interpreter/game
room is active, it can corrupt state and cause bugs that look like rendering
failures.

## Duplicate Instances

Do not create an object in code if the room already contains it unless the
project deliberately wants multiple instances.

Check both:

- room `.yy` placed instances
- `instance_create_*` calls in Create/Step/bootstrap code

If an object is persistent and also placed in its home room, returning to that
room can create duplicate state unless the project guards against it.

Useful diagnostic:

```gml
show_debug_message("room=" + room_get_name(room)
    + " editors=" + string(instance_number(obj_editor))
    + " interpreters=" + string(instance_number(obj_basic_interpreter)));
```

## Room Transitions

Room bugs often masquerade as black screens, input failures, or missing UI.

When investigating room transitions, log:

- current room before transition
- target room
- return room global
- current mode global
- relevant instance counts
- running/ended state flags

Example:

```gml
show_debug_message("RETURN: room=" + room_get_name(room)
    + " ret=" + room_get_name(global.editor_return_room)
    + " running=" + string(global.interpreter_running)
    + " ended=" + string(global.program_has_ended));
```

If a screen goes black, do not assume a black rectangle is covering the UI.
First prove which room you are in and whether the active room's Draw event is
exiting early.

## Return Room State

Be careful with globals such as `global.editor_return_room`,
`global.previous_room`, or `global.resume_room`.

Bad pattern:

```gml
global.editor_return_room = rm_editor;
// many lines later
global.editor_return_room = room;
```

This can silently poison the return target if the function is called from the
wrong room by a persistent object.

Better pattern:

```gml
global.editor_return_room = rm_editor;
```

Or, if the caller room really matters:

```gml
if (room == rm_editor) {
    global.editor_return_room = room;
}
```

## Web, HTML5, And Opera GX Targets

Do not assume all web exports report the same runtime constants.

Observed problem patterns:

- `operagx` is often the practical CLI target even when people call the result
  "HTML5."
- `html5` may not be supported by the installed `gm-cli`.
- `os_browser` may be `browser_not_a_browser` under some web export paths.
- `os_type` may not be the value an agent expects from online examples.

Always verify the actual target used by the deploy script.

For web-specific behavior, prefer a project-local helper such as:

```gml
function project_is_web_runtime() {
    return (os_type == os_html5)
        || (os_type == os_gxgames)
        || (os_browser != browser_not_a_browser);
}
```

Then use the helper consistently. If a project already has a helper, use it
rather than scattering new platform checks.

## GUI Coordinates

Draw GUI and room coordinates are not the same thing.

If an object draws controls in Draw GUI, read input in GUI coordinates:

```gml
var gx = device_mouse_x_to_gui(0);
var gy = device_mouse_y_to_gui(0);
```

Do not pair Draw GUI rendering with `device_mouse_x()` / `device_mouse_y()`
unless the project has proven that room coordinates match GUI coordinates.

Also guard display GUI size when needed:

```gml
var gw = display_get_gui_width();
var gh = display_get_gui_height();
if (gw <= 0) gw = room_width;
if (gh <= 0) gh = room_height;
```

## Surfaces And Runtime Assets

Surfaces and runtime-created sprites can be lost or invalidated.

For surfaces:

- create them in a known owner object
- recreate if `!surface_exists(surface_id)`
- free them in Destroy only if ownership is clear
- reset the render target after drawing

For runtime sprites:

- delete old sprite assets before replacing them
- rebuild if `!sprite_exists(sprite_id)`
- clear references after `sprite_delete`

Never call GameMaker runtime functions with unchecked invalid IDs if the input
can come from a user program or external data.

## Draw Order, Depth, And Backgrounds

If something exists but is invisible, inspect draw order before rewriting the
rendering logic.

Check:

- room background layers
- instance layer depths
- `instance_create_depth` values
- object Draw events versus Draw GUI events
- full-screen clears or rectangles drawn later in the pipeline

Depth mistakes can make a correct object draw behind an opaque background. This
is especially easy when adding code-created visual helpers such as habitat
renderers, overlays, grids, or debug views to an existing room.

## State Reset Between Runs

Interpreter-like projects need explicit reset paths. Do not rely on room
restarts to clean everything.

Reset likely includes:

- running/ended flags
- input mode flags
- pause flags
- queues
- stacks
- line pointers
- current mode
- open file handles
- generated sounds
- runtime sprites/surfaces
- temporary instances
- open diagnostic/log files

If a bug only happens on the second run, suspect leaked state first.

## Toggle Diagnostic Systems

Forensic logging, snapshot exporters, auto-exit test loops, debug overlays, and
similar diagnostics are useful while tuning, but they should not become default
game behavior.

When adding diagnostics:

- put them behind a named setting such as `forensics_enabled`
- default the setting off unless the project is explicitly a test harness
- avoid opening files when the setting is off
- guard Step/Draw code so disabled diagnostics do nothing safely
- make auto-exit behavior conditional on the diagnostic setting
- show clear on-screen status such as `forensics: off` when useful

This keeps verification tools available without surprising the user during
normal play.

## Input Handling

Input bugs often come from multiple objects reading the same key in the same
frame.

Check:

- persistent objects processing input in the wrong room
- duplicate input-handler instances
- `keyboard_string` not cleared after consuming text
- Enter being consumed by both an overlay and a base editor
- virtual keyboard and hardware keyboard both feeding the same queue

Pattern for modal overlays:

```gml
if (showing_overlay) {
    // handle overlay keys
    exit; // block base input
}
```

## Debugging Strategy For LLM Agents

When stuck, stop editing and instrument.

Use temporary, targeted logs around:

- room names
- object counts
- key global flags
- event entry points
- resource IDs
- target constants

Keep logs narrow. A thousand lines of unfocused debug output is another way to
get lost.

### Runtime Evidence Hierarchy

Trust verification sources in this order:

1. **Project transcript or assert file** (`TEST: ... = PASS|FAIL`) written for
   automation — see `LLM_VERIFICATION_HARNESS_PATTERN.md`.
2. **Screenshot or capture** at a path documented in the project's workflow.
3. **Targeted** `show_debug_message` lines tied to one hypothesis.
4. **Igor / compiler output** for build failures and crashes.
5. **Source inspection alone** — lowest confidence for behavior bugs.

Do not declare success from exit code alone if the Runner waits for a keypress
after the game ends. See `GM_CLI_HEADLESS_VERIFY.md`.

## Build And Verify

Use the project's own build/deploy scripts when they exist.

### gm-cli And Headless Runs

On LTS with GameMaker CLI installed:

```powershell
gm-cli run "Project.yyp" --target windows --no-errors-only
```

First run may download runtimes (minutes). Allow long timeouts; poll the
project's transcript file rather than assuming failure at 60 seconds.

Full detail: `GM_CLI_HEADLESS_VERIFY.md`.

### File-Based Verification

If the project documents `docs/AUTOTEST_WORKFLOW.md` (or similar):

1. Copy a committed test from `tests/` or `diagnostics/` to the runtime
   trigger path.
2. Run via `gm-cli` or IDE.
3. Read the transcript path — grep for `TEST: .* = FAIL`.
4. For `SCREENSHOT=REQUESTED`, inspect the documented PNG path.

Pattern reference: `LLM_VERIFICATION_HARNESS_PATTERN.md`. Optional helper:
`tools/agent-verify.ps1.template`.

### Web Builds

For web builds:

- verify with the same target the script uses
- cache-bust deployed URLs when testing
- capture console logs
- visually verify canvas output
- test at least one real interaction, not just page load

If `gm-cli package --target html5` fails but the project deploys with
`--target operagx`, do not fight the toolchain. Use the working project script
unless the user asked to change build targets.

## Generated Output Locks And Toolchain Failures

Generated folders such as `output/`, `_cache/`, and `_temp/` can be locked by
the IDE, a previous run, antivirus, a network filesystem, or a stuck helper
process. A cleanup/package failure in those folders is not automatically a GML
failure.

When verification fails:

- read the error carefully and distinguish source serialization, compilation,
  package creation, and runtime execution
- check for lingering `Igor`, game, or `GMAssetCompiler` processes
- use the project's known-good command when available
- report generated-output locks or compiler permission failures separately from
  code errors
- avoid destructive cleanup of generated folders unless the target path is
  verified and the user has approved or the project convention permits it

Stage and commit source changes deliberately. IDE metadata churn and generated
output noise should not be mixed with behavior changes unless those files are
actually part of the fix.

## Safe Change Discipline

Before changing code, state the suspected cause in one sentence.

After changing code, verify that exact cause.

Avoid broad rewrites when a one-line guard or metadata fix would prove the
bug. GameMaker projects often contain many interdependent event paths; broad
"cleanup" can create fresh failures.

## Adapting External Algorithms And Repositories

Codex, Claude Code, and similar coding agents can use public repositories as
implementation references, but discovery is not permission to copy blindly.

Before adapting an external algorithm:

- inspect the repository license and record the source URL and revision
- distinguish a concept-level reimplementation from substantially copied code
- preserve copyright and license notices when the license requires them
- avoid importing an entire application when a small, testable algorithm port
  fits the existing GameMaker architecture
- isolate generation, simulation data, and rendering so each can be verified
  independently
- add a stored seed and fixed-seed checks for procedural systems
- introduce the new path incrementally, with a comparison switch when practical
- document deliberate differences caused by grid shape, performance, or game
  design

For connected procedural graphics such as roads, rivers, walls, or coastlines,
derive topology from game data first. A four-neighbor bitmask provides 16
endpoint, straight, bend, junction, and crossing cases. Generated artwork can
provide texture and style, but it should not decide which edges connect.

Keep attribution and implementation notes in project-local documentation. Add
only reusable workflow lessons to this shared guide.

## What To Add To This Guide

Add a new lesson when:

- the mistake is GameMaker/GML-specific
- it is likely to recur in other projects
- it can be written as a concrete rule or diagnostic
- it is not just a local project preference

Keep project-specific details in that project's own docs. Shared examples can
be included here only when anonymized and broadly useful.
