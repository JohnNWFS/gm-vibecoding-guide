# Best Practices For LLM Vibecoding For GML

This guide is for Codex, Claude Code, ChatGPT, and other LLM agents working on
GameMaker Studio / GML projects. It supplements official GameMaker
documentation with field-tested practices that prevent common agent failures.

The purpose is simple: move carefully, prove runtime state, and avoid the
GameMaker-specific traps that make an agent chase symptoms for an hour.

## How To Use This File

Before changing a GameMaker project:

1. Read this file.
2. Read the project's own docs, especially any LLM brief, status file, or
   autotest workflow.
3. Inspect the relevant `.yy` metadata as well as the `.gml` code.
4. Form one testable hypothesis.
5. Add the smallest safe change.
6. Verify in the actual target runtime.

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

If a bug only happens on the second run, suspect leaked state first.

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

## Build And Verify

Use the project's own build/deploy scripts when they exist.

For web builds:

- verify with the same target the script uses
- cache-bust deployed URLs when testing
- capture console logs
- visually verify canvas output
- test at least one real interaction, not just page load

If `gm-cli package --target html5` fails but the project deploys with
`--target operagx`, do not fight the toolchain. Use the working project script
unless the user asked to change build targets.

## Safe Change Discipline

Before changing code, state the suspected cause in one sentence.

After changing code, verify that exact cause.

Avoid broad rewrites when a one-line guard or metadata fix would prove the
bug. GameMaker projects often contain many interdependent event paths; broad
"cleanup" can create fresh failures.

## What To Add To This Guide

Add a new lesson when:

- the mistake is GameMaker/GML-specific
- it is likely to recur in other projects
- it can be written as a concrete rule or diagnostic
- it is not just a local project preference

Keep project-specific details in that project's own docs. Shared examples can
be included here only when anonymized and broadly useful.

