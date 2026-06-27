# GameMaker CLI Headless Verification

This guide is for LLM agents and developers using **GameMaker LTS** with
**`gm-cli`** to compile and run a project without opening the IDE.

It complements `BEST_PRACTICES_FOR_LLM_VIBECODING_FOR_GML.md`. Pair it with
`LLM_VERIFICATION_HARNESS_PATTERN.md` when the project writes machine-readable
test output to disk.

## Why Use gm-cli

Agents need a repeatable loop:

1. Edit GML / `.yy` metadata.
2. Compile and run from the shell.
3. Read a **small verification artifact** (transcript, log, screenshot path).
4. Fix and repeat.

Opening the IDE for every change is slow and hard to automate. `gm-cli` is the
practical headless entry point on modern LTS installs.

## Basic Commands

From the project directory (paths are examples — use your `.yyp` name):

```powershell
# Compile and run Windows VM (common for desktop debugging)
gm-cli run "MyProject.yyp" --target windows

# Quieter compile log (still verbose at runtime)
gm-cli run "MyProject.yyp" --target windows --no-errors-only

# Package for web (target varies by installed runtimes — see below)
gm-cli package "MyProject.yyp" --target operagx -o build.zip
```

First run on a machine may **download runtimes and tools** (several minutes).
Agents should run the command in the background and poll for completion or for
a project-defined transcript file — not abandon after 60–120 seconds.

## Where Igor Lives

After install, tooling is typically under:

```text
%LOCALAPPDATA%\GameMakerCLI\cache\runtimes-gms2\runtime-<version>\
```

Igor and Runner paths vary by version. Do not hard-code one runtime folder in
shared docs; discover it from a successful `gm-cli` run or the project's local
notes.

## stdout Is Not Your Test Result

`gm-cli run` streams **compiler output**, **engine boot logs**, and often
**every `show_debug_message` line** into the same console. A single run can
produce thousands of lines.

Rules for agents:

- Do **not** treat full Igor stdout as the pass/fail signal.
- Do **not** ask the user to paste the whole console unless debugging compile
  failures.
- Prefer a **project-defined transcript or snapshot file** (see
  `LLM_VERIFICATION_HARNESS_PATTERN.md`).
- Use stdout only for **compile errors** and **crash signatures**.

## Exit Codes Are Unreliable

A common false failure: the game runs correctly, prints pass lines to a
transcript, then the Runner **waits for a keypress** (`ESC`, `Enter`, etc.).
The agent kills or closes the process; Igor reports exit code `1` or
`4294967295` (`-1`).

**Treat exit code as unreliable** when:

- The game shows an end-of-run message and blocks on input.
- The agent did not use a non-interactive quit path documented by the project.
- A transcript file exists and shows success.

Verification priority:

1. Transcript / assert file (if the project provides one).
2. Screenshot or saved capture (for visual tests).
3. Compile succeeded + targeted debug lines.
4. Igor exit code (lowest trust for interactive runners).

## Typical Agent Loop (Windows PowerShell)

```powershell
# 1. Stage trigger input (project-specific path — see project docs)
Copy-Item -Force "tests\my_scenario.dat" "$env:USERPROFILE\MyGameSave\autotest.dat"

# 2. Clear old transcript if the project documents one
$transcript = "$env:USERPROFILE\MyGameSave\autotest_output.txt"
if (Test-Path $transcript) { Remove-Item $transcript -Force }

# 3. Run (allow several minutes on cold cache)
gm-cli run "MyProject.yyp" --target windows --no-errors-only

# 4. Read transcript — not exit code alone
Get-Content $transcript -Raw
```

Projects should document exact trigger and transcript paths in
`docs/AUTOTEST_WORKFLOW.md` or equivalent.

## Parsing Pass / Fail

If the project uses an assert convention in the transcript:

```text
TEST: FEATURE_X = PASS
TEST: FEATURE_Y = FAIL
```

PowerShell example:

```powershell
$text = Get-Content $transcript -Raw
if ($text -match 'TEST: .* = FAIL') { throw "Test failures found" }
```

Or use the template script `tools/agent-verify.ps1.template` in this guide repo.

## Compile Failures vs Runtime Failures

| Symptom | Likely layer |
|--------|----------------|
| Error before `Run game` / `Entering main loop` | Compile, missing `.yy` entry, syntax |
| Compile OK, black screen, no transcript | Room/instance/metadata (see best practices) |
| Transcript shows `FAIL` | Game logic / feature bug |
| Transcript all `PASS`, Igor exit non-zero | Harness / keypress wait — often not a bug |

## Web / Package Targets

Do not assume `html5` is available. Many LTS + `gm-cli` setups use **`operagx`**
or another export the project already documents.

Always use the **same target as the project's deploy script**. If
`gm-cli package --target html5` fails but `operagx` works, follow the project;
do not fight the toolchain unless the user asked to change targets.

## Windows Shell Notes

- **PowerShell does not support bash heredocs** for `git commit`. Use:
  `git commit -m "title" -m "body"`.
- Long paths: quote the `.yyp` path.
- Antivirus or IDE locks on `.gmcache` / `output/` can break package steps —
  distinguish **permission/lock errors** from GML bugs (see best practices).

## What Belongs In This Shared Guide

Generic `gm-cli` behavior, exit-code traps, and agent discipline.

## What Belongs In Each GameMaker Project

- Exact transcript and trigger file paths.
- Bootstrap object/event names.
- Screenshot output paths.
- Copy-paste one-liner for that repo's verify loop.
- Optional wrapper: `tools/agent-test.ps1` copied from this guide's template.

Use `PROJECT_AUTOTEST_TEMPLATE.md` when adding harness docs to a new project.