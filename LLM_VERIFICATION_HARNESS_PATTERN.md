# LLM Verification Harness Pattern

A **project-neutral** pattern for letting Codex, Claude, ChatGPT, and other
agents verify GameMaker games **without a human watching the Runner window**.

The IDE and Igor console are poor interfaces for agents: too much noise, no
stable pass/fail signal, and interactive keypress waits that poison exit codes.
Instead, the **game writes a small machine-readable artifact** each run.

This document describes how to design that harness. Project-specific paths and
bootstrap code live in each repo's `docs/AUTOTEST_WORKFLOW.md` (or similar).

See also: `GM_CLI_HEADLESS_VERIFY.md`, `PROJECT_AUTOTEST_TEMPLATE.md`,
`tools/agent-verify.ps1.template`.

## Goals

- Agent can run `gm-cli` (or the IDE) and **read one file** to know pass/fail.
- Visual features have an optional **screenshot or capture path**.
- Committed tests live in a **`tests/` or `diagnostics/`** folder; runtime
  trigger file is a copy, not the only source.
- Harness is **off by default** when no trigger file is present.

## Architecture

```text
  tests/feature_smoke.dat          (committed repro)
           |
           v  copy at verify time
  <runtime trigger file>           (user save dir or datafiles/)
           |
           v  bootstrap on Step (not Create)
  run scenario / playtest hook
           |
           v  single commit function
  <transcript file>                (rewritten each run)
           ^
           |
     agent reads this
```

## Components

### 1. Trigger file

A file whose **existence** (or content flag) means "autorun this scenario on
launch." Examples:

- `autotest.json` in a known save directory
- `autotest.dat` / `autotest.bas` / `smoke_test.gml` per project convention
- A `global.autotest_requested` set by reading a datafile at boot

The trigger should be easy to copy from a committed test:

```powershell
Copy-Item -Force "tests\smoke_level.dat" "<trigger path documented in project>"
```

### 2. Bootstrap timing

Run the harness from **Step** (or a dedicated controller's Step), **not
Create** on the first object that boots, unless you have proven all globals
exist in Create.

Common failure: bootstrap in Create runs before `global.config`, save paths, or
audio init — silent no-op or crash.

Guard bootstrap:

```gml
if (!variable_global_exists("config_ready") || !global.config_ready) return false;
if (global.autotest_bootstrapped) return false;
global.autotest_bootstrapped = true;
// load trigger, start scenario
```

### 3. Transcript file

One text file, **truncated at the start of each test run**, appended during
the scenario, finalized on completion.

**Header** (parseable by agents):

```text
# <PROJECT> AUTOTEST TRANSCRIPT
# MODE=TEXT
# SCREENSHOT=OPTIONAL
```

For visual tests, flip header when a marker is detected in the trigger:

```text
# MODE=SCREENSHOT
# SCREENSHOT=REQUESTED
```

**Footer** (proves the run reached end):

```text

Program has ended - ESC or ENTER to return
```

Use wording the project controls; agents grep for a stable footer string.

### 4. Single commit path

Route all **agent-relevant text output** through one function:

```gml
/// Appends to on-screen log AND transcript when autotest is active.
function verify_transcript_append(_line) {
    // ds_list_add(global.output_lines, _line);  // if applicable
    if (!global.autotest_transcript_enabled) return;
    var f = file_text_open_append(global.autotest_transcript_path);
    file_text_write_string(f, _line);
    file_text_writeln(f);
    file_text_close(f);
}
```

Without centralization, agents miss output that only hits `draw_text` or
`show_debug_message`.

### 5. Assert convention

Print lines agents can count:

```text
TEST: PLAYER_SPAWN = PASS
TEST: ENEMY_AI = FAIL
```

Convention:

- Prefix: `TEST:`
- Name: `UPPER_SNAKE`
- Result: `PASS` or `FAIL`
- Optional: `TEST: SUMMARY 4/4 PASS`

Avoid prose-only success messages for automated runs.

### 6. Screenshot / capture marker

Pure Draw / tile / surface output **does not** appear in a text transcript.
Support an explicit marker in the trigger payload:

```text
AUTOTEST_SCREENSHOT
```

When seen, the harness:

1. Sets transcript header to `SCREENSHOT=REQUESTED`.
2. Saves a PNG to a **documented path** (e.g. next to the transcript).
3. Optionally exits without requiring human inspection of the window.

Document the PNG path in the project workflow doc.

### 7. Committed tests folder

Keep repros in the repo:

```text
tests/
  combat_smoke.dat
  inventory_roundtrip.dat
diagnostics/          # alternate name used by some projects
  mode2_grid_smoke.dat
```

Agent workflow:

1. Copy one file to the trigger path.
2. Run `gm-cli`.
3. Read transcript (and screenshot if requested).
4. Never edit only the runtime copy without updating the committed test.

## Disable Harness

Normal play: **no trigger file** → game starts normally.

Document how to disable: delete/rename trigger file, or unset a project flag.

## Anti-Patterns

| Anti-pattern | Why it fails |
|--------------|--------------|
| Agent reads only Igor stdout | Noise, truncation, no PASS/FAIL |
| Trust Igor exit code after success footer | Keypress wait → false failure |
| Bootstrap in Create before globals | Race / silent skip |
| Visual test with no screenshot path | Agent claims "looks fine" |
| New script without `.yy` + `.yyp` entry | Compile or link miss |
| One-off test only in save dir, not in repo | Lost on next machine |

## Runtime Evidence Hierarchy

When verifying a change, trust sources in this order:

1. **Transcript file** with `TEST:` lines (automation-first).
2. **Screenshot / recording** at documented path.
3. **Targeted** `show_debug_message` around the hypothesis.
4. Full Igor log (compile errors, crashes).
5. Source-code reasoning alone (lowest).

## Minimal GML Checklist For New Harness

- [ ] `global.autotest_transcript_enabled` default false
- [ ] Transcript path in one function (easy to document)
- [ ] Reset transcript at run start
- [ ] Finalize transcript before "press key to exit"
- [ ] `global.autotest_bootstrapped` guard
- [ ] Step-event bootstrap after config ready
- [ ] Project doc with trigger path, transcript path, screenshot path
- [ ] One committed smoke test in `tests/` or `diagnostics/`
- [ ] Optional: copy of `tools/agent-verify.ps1.template` in project `tools/`

## Adopting In An Existing Project

1. Copy `PROJECT_AUTOTEST_TEMPLATE.md` into `docs/AUTOTEST_WORKFLOW.md`.
2. Fill in paths and bootstrap object names.
3. Add `verify_transcript_append()` (or extend existing output commit).
4. Add one smoke test with two or three `TEST:` lines.
5. Document the `gm-cli` one-liner in the project README or LLM brief.
6. Link to this guide from `PROMPT_FOR_NEW_GAMEMAKER_PROJECTS.md` in the
   project README for future agents.

Keep game-specific scenario content in the game repo. Keep this file generic.