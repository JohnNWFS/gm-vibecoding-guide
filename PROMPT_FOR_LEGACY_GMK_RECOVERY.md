# Prompt For Legacy GMK Recovery Projects

Paste this into a new Codex, Claude Code, or ChatGPT coding session when trying
to recover an ancient pre-GameMaker-Studio project such as `.gmk`, `.gm81`,
`.gmd`, or `.gb1` and rebuild it in modern GameMaker LTS.

```text
You are helping recover and modernize an ancient pre-GameMaker-Studio project
into a modern GameMaker LTS project.

Before editing anything, read the shared GameMaker LLM guide:

C:\Users\hoffe\GameMakerProjects\_GM_VibeCoding_Guide\BEST_PRACTICES_FOR_LLM_VIBECODING_FOR_GML.md

Then inspect the local folder for:
- all `.gmk`, `.gm81`, `.gmx`, `.gb1`, `.gmd`, `.exe`, `.zip`, `.rar`, `.7z`
- alternate versions, backups, numbered copies, and possible less-corrupt siblings
- extracted assets, screenshots, sound files, notes, readmes, and old exported builds

Goal:
Recover enough information from the old project to rebuild a modern GameMaker
LTS version that preserves the original intent, behavior, assets, and feel
where possible.

Work in phases:

1. Evidence pass
   - Do not assume the first legacy file is the best source.
   - Inventory every candidate version.
   - Record file sizes, dates, names, and likely relationships.
   - Try safer, read-only inspection before conversion attempts.
   - If import fails, inspect alternate copies before concluding the project is unrecoverable.

2. Extraction pass
   - Try available GameMaker legacy tools/importers where possible.
   - If import fails, attempt asset/code extraction from the legacy file or sibling files.
   - Extract or identify sprites, backgrounds, sounds, object names, room names, scripts, and any readable GML/event text.
   - Use binary/string inspection if necessary, but do not overwrite the original files.
   - Save extracted artifacts into a clearly named recovery folder.

3. Interpretation pass
   - Summarize what the program appears to be.
   - Identify the core loop, visible actors, controls, resources, scoring/state, and likely player/simulation goal.
   - Separate confirmed facts from informed guesses.
   - Include screenshots or asset previews if useful.

4. Reconstruction pass
   - Create or use a modern blank GameMaker LTS project.
   - Put the current state in git before substantial edits.
   - Rebuild objects, rooms, assets, events, and behavior incrementally.
   - Treat `.yy`, `.yyp`, room instance data, event metadata, and resource order as code.
   - Use project-local patterns and avoid broad rewrites.
   - Avoid GML built-in variable/function names for locals or instance variables.
   - Keep old artifacts available for comparison.

5. Verification pass
   - Use the installed GameMaker CLI/Igor or project-specific build command when available.
   - Run a built-in-name scan before compiling.
   - Verify that the game loads and the main behavior runs.
   - If the simulation/game needs diagnostics, add a toggleable logger or snapshot exporter, defaulted off.
   - If the build system fails due to generated output locks or toolchain permission issues, report that separately from GML/runtime failures.

6. Teaching/documentation pass
   - Create a short recovery report:
     - source files inspected
     - extraction methods attempted
     - confirmed recovered assets/code
     - inferred original design
     - modern reconstruction choices
     - known deviations from original
     - exact validation commands and results
   - If any lesson is broadly useful to GameMaker recovery work, recommend an update to the shared guide.
   - Keep project-specific quirks in local docs.

Important:
- Do not discard user/IDE changes unless explicitly asked.
- Do not treat generated `output/`, cache folders, or IDE metadata churn as source unless proven relevant.
- If multiple versions exist, compare them instead of betting everything on the first file.
- If import fails, the task is not over; asset extraction plus behavioral reconstruction is still useful.
```
