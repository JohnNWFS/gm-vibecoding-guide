# Project Autotest Workflow

> **Template:** Copy this file into your GameMaker project as
> `docs/AUTOTEST_WORKFLOW.md` and replace every `<!-- ... -->` placeholder.
> Do not commit game-specific paths to the shared
> [gm-vibecoding-guide](https://github.com/JohnNWFS/gm-vibecoding-guide) repo.

This project supports a launch-time autotest hook so LLM agents and CI can
verify behavior without parsing the Igor console.

## Files And Paths

| Role | Path |
|------|------|
| Committed tests | `<!-- e.g. tests/ or diagnostics/ -->` |
| Runtime trigger file | `<!-- e.g. %USERPROFILE%\MyGame\autotest.dat -->` |
| Text transcript | `<!-- e.g. %USERPROFILE%\MyGame\autotest_output.txt -->` |
| Screenshot (optional) | `<!-- e.g. %USERPROFILE%\MyGame\autotest_capture.png -->` |
| Bootstrap script | `<!-- e.g. scripts/autotest_bootstrap/autotest_bootstrap.gml -->` |
| Transcript helper | `<!-- e.g. scripts/verify_output/verify_output.gml -->` |
| Bootstrap call site | `<!-- e.g. objects/obj_game_controller/Step_0.gml -->` |
| Transcript reset | `<!-- e.g. scripts/game_start/game_start.gml -->` |

## Launch Behavior

When `<!-- trigger filename -->` exists at the documented path:

1. <!-- Describe load step -->
2. <!-- Describe run/scenario start -->
3. Transcript file is deleted/recreated at run start.
4. Committed output is appended via `<!-- commit function name -->`.
5. On completion, transcript receives footer:
   `<!-- exact footer string -->`

If the trigger file is absent, the game starts normally.

## Transcript Format

```text
# <!-- PROJECT NAME --> AUTOTEST TRANSCRIPT
# MODE=TEXT
# SCREENSHOT=OPTIONAL

<!-- example output lines -->

<!-- footer line -->
```

### Screenshot mode

Include this marker in the trigger payload when visual verification is required:

```text
AUTOTEST_SCREENSHOT
```

Header becomes:

```text
# MODE=SCREENSHOT
# SCREENSHOT=REQUESTED
```

Agents should read the PNG at the screenshot path above.

## Assert Convention

Tests should print:

```text
TEST: <!-- NAME --> = PASS
TEST: <!-- NAME --> = FAIL
```

Agents fail the run if any line matches `TEST: .* = FAIL`.

## Implementation Notes

- Bootstrap runs from **Step**, not Create, after `<!-- prerequisite global -->` exists.
- Use `global.autotest_bootstrapped` (or equivalent) to run once per session.
- Route agent-visible text through **one** commit/transcript function.
- Draw-only / tile-only output does not appear in the text transcript; use
  screenshot mode for those tests.

## Agent Verify Loop

```powershell
# From project root — adjust paths to match this doc
$trigger = "<!-- full trigger path -->"
$transcript = "<!-- full transcript path -->"
Copy-Item -Force "tests\<!-- smoke file -->" $trigger
if (Test-Path $transcript) { Remove-Item $transcript -Force }
gm-cli run "<!-- Project.yyp -->" --target windows --no-errors-only
Get-Content $transcript -Raw
```

Or use `tools/agent-test.ps1` if this project copied
`tools/agent-verify.ps1.template` from the vibecoding guide.

## Disable Autotest

<!-- e.g. Delete or rename the trigger file. -->

## See Also

- [GM_CLI_HEADLESS_VERIFY.md](https://github.com/JohnNWFS/gm-vibecoding-guide/blob/main/GM_CLI_HEADLESS_VERIFY.md)
- [LLM_VERIFICATION_HARNESS_PATTERN.md](https://github.com/JohnNWFS/gm-vibecoding-guide/blob/main/LLM_VERIFICATION_HARNESS_PATTERN.md)