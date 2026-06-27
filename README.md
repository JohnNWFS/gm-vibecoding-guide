# GameMaker LLM Vibecoding Guide

Shared guidance for using Codex, Claude Code, ChatGPT, Grok, and other LLM
coding agents on **GameMaker Studio / GML** projects (including LTS).

The guide is intentionally **project-neutral**. Keep game-specific paths,
bootstrap code, and scenario tests in each project's own docs. Add only broadly
reusable GameMaker/GML lessons here.

## Start Here

| Doc | Purpose |
|-----|---------|
| [BEST_PRACTICES_FOR_LLM_VIBECODING_FOR_GML.md](BEST_PRACTICES_FOR_LLM_VIBECODING_FOR_GML.md) | Metadata, rooms, persistence, web targets, GML traps |
| [GM_CLI_HEADLESS_VERIFY.md](GM_CLI_HEADLESS_VERIFY.md) | `gm-cli` runs, exit-code traps, agent verify loop |
| [LLM_VERIFICATION_HARNESS_PATTERN.md](LLM_VERIFICATION_HARNESS_PATTERN.md) | File-based transcripts, `TEST:` asserts, screenshots |
| [PROMPT_FOR_NEW_GAMEMAKER_PROJECTS.md](PROMPT_FOR_NEW_GAMEMAKER_PROJECTS.md) | Paste-in prompt for new sessions |
| [PROMPT_FOR_LEGACY_GMK_RECOVERY.md](PROMPT_FOR_LEGACY_GMK_RECOVERY.md) | Ancient `.gmk` / `.gm81` recovery |
| [PROJECT_AUTOTEST_TEMPLATE.md](PROJECT_AUTOTEST_TEMPLATE.md) | Copy into a project as `docs/AUTOTEST_WORKFLOW.md` |
| [PROJECT_LESSONS_TEMPLATE.md](PROJECT_LESSONS_TEMPLATE.md) | Per-project lessons log |
| [LESSONS_LEARNED_FROM_LONG_TERM_CHATS.md](LESSONS_LEARNED_FROM_LONG_TERM_CHATS.md) | Collaboration patterns (ChatGPT-era) |

## Tools

| File | Purpose |
|------|---------|
| [tools/agent-verify.ps1.template](tools/agent-verify.ps1.template) | Copy to a project as `tools/agent-test.ps1`; set paths |

## Quick Path For Agents

1. Read **Best Practices** + **GM CLI** + **Harness Pattern**.
2. In the game repo, find `docs/AUTOTEST_WORKFLOW.md` or copy **Project Autotest Template**.
3. Copy a committed test → run `gm-cli` → read transcript (not Igor exit code alone).

## Contributing

Add lessons here when they are GameMaker/GML-specific, likely to recur, and
not tied to one game's content. Keep scenario files and save-directory paths in
the game repository.