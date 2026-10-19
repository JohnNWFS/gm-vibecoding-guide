# Headless Linux Verification For Cloud Agents

This guide is for running a GameMaker LTS project's runtime tests on **Linux
with no display and no IDE**: a cloud coding agent's sandbox (Claude Code on
the web, Codex cloud, etc.), a CI runner, or a headless server.

It extends `GM_CLI_HEADLESS_VERIFY.md` (Windows / `gm-cli`) and assumes the
project already has a file-based harness from
`LLM_VERIFICATION_HARNESS_PATTERN.md`. Without that harness there is nothing
to read back on a machine with no screen.

Verified in October 2026 with GameMaker LTS 2026 (runtime 2026.0.0.23),
`@gamemaker/gm-cli` 2.4.1, Ubuntu 24.04, VM target.

## Why Bother

A cloud agent can edit GML all day but, without this, cannot prove the game
still compiles or that tests pass. The owner then has to run the Windows
script after every change. With a Linux build, the agent can do the full loop
itself: edit, compile, run, read the transcript, fix.

## The Short Version

1. Network allowlist: `gamemaker.io`, `*.gamemaker.io`, `gms.yoyogames.com`,
   plus the npm registry and GitHub.
2. Sign in with an **access key** (from
   `https://gamemaker.io/en/account/access-keys`); this also downloads Igor.
3. Install the LTS runtime with **Igor directly**.
4. Compile with Igor's `linux Compile`. **Not** `gm-cli compile`.
5. Unzip the game plus the runtime's stock `runner`, add OpenSSL 1.0.
6. Stage the test trigger, run under `xvfb-run`, read the transcript.

A complete, copyable script: `tools/linux-ci.sh` in
[adventure-arena](https://github.com/JohnNWFS/adventure-arena) (project-neutral
apart from the transcript format).

## Trap 1: gm-cli Compile Is Blocked From Cloud IPs

`gm-cli compile` / `run` first downloads a **project-tool** package from
`gmpm.gamemaker.io`. Cloudflare in front of that host returns
**HTTP 403 "Attention Required"** to many datacenter IP ranges. Adding the
domain to the sandbox allowlist does not help; the block is on GameMaker's
side.

```text
npm error 403 403 Forbidden - GET https://gmpm.gamemaker.io/@gm-tools%2fproject-tool-linux-x64
```

Igor does not need project-tool to compile a normal project. Skip gm-cli for
the build and call Igor directly. Igor, the runtime feed and the runtime zips
all come from `gms.yoyogames.com`, which is not blocked.

Rules for agents:

- A 403 HTML page from `gmpm` is **not** a bad key or a missing allowlist entry.
  Do not loop on retries or ask the owner to add more domains for it.
- Use gm-cli only for `login`.

## Trap 2: Getting A License Non-Interactively

```bash
NODE_USE_ENV_PROXY=1 npx --yes @gamemaker/gm-cli@2.4.1 login "$GM_ACCESS_KEY" --print > raw.txt
```

- `login` refuses to run outside a project folder unless you pass `--print`.
- `--print` writes **progress text and ANSI codes to stdout** ahead of the
  plist. Strip it, or Igor silently ignores the "license":
  ```bash
  sed 's/\x1b\[[0-9;]*[A-Za-z]//g' raw.txt | sed -n '/<plist/,/<\/plist>/p' > licence.plist
  chmod 600 licence.plist
  ```
- Check for `<string>success</string>` in the result.
- `login` also downloads Igor into gm-cli's cache (`/tmp/gm-cli-cache/igor/...`
  by default; pass `--cache-dir` to control it). Igor can also be fetched
  directly from `https://gms.yoyogames.com/igor_linux-x64.zip`.
- **Behind an HTTPS proxy** (most agent sandboxes): Node's built-in `fetch`
  ignores `HTTPS_PROXY` and fails with a bare `fetch failed`. Set
  `NODE_USE_ENV_PROXY=1` (Node 22.21 or later).

Security rules:

- Read the key from an environment variable. Never commit it or the plist.
- Make a dedicated key for the sandbox and revoke it when finished; agent
  transcripts and sandbox disks are not secret stores.
- Never print the plist; it contains the account email and token.

## Trap 3: Installing The Runtime

```bash
Igor --lf=licence.plist --ru=https://gms.yoyogames.com/Zeus-Runtime-LTS2026.rss \
     --rp=$WORK/runtimes -m=linux -- runtime Install
```

- `-m=linux` keeps the download to the modules you need.
- "Verification Inconclusive no manifest found" at the end is harmless.
- The feed URL selects the release channel. Match the IDE the owner uses
  (`Zeus-Runtime-LTS2026.rss` for LTS 2026).

## Trap 4: Use `Compile`, Not `Run` Or `PackageZip`

| Igor command (`-- linux <cmd>`) | Result in a sandbox |
|---|---|
| `PackageZip` | Not a Linux command; Igor lists valid ones and exits |
| `Package` | Compiles, then throws `FileNotFoundException` moving the zip |
| `Run` | Compiles, then fails calling `linuxdeploy` to build an AppImage |
| **`Compile`** | **Writes `<of>.zip` containing `assets/game.unx` and data files** |

```bash
Igor --rp=$RUNTIME --project=MyGame.yyp --cache=$WORK/cache --temp=$WORK/temp \
     --of=$WORK/out/MyGame --lf=licence.plist --runtime=VM -- linux Compile
```

Then assemble a runnable folder from the compile zip and the runtime's own
runner:

```bash
unzip -qo $WORK/out/MyGame.zip -d stage
unzip -qo $RUNTIME/linux/runner.zip -d stage   # a single binary named "runner"
chmod +x stage/runner
```

The runner loads `assets/game.unx` relative to its own folder.

Compile health check: the log line `Stats : GMA : sp=...,au=...,ob=...` gives
resource counts. Compare sprite/sound/object counts with what the project
should have.

## Trap 5: The Runner Needs OpenSSL 1.0 And libGLU

`ldd stage/runner` on a current distro shows:

```text
libcrypto.so.1.0.0 => not found
libssl.so.1.0.0    => not found
libGLU.so.1        => not found
```

- `libGLU`: `apt-get install libglu1-mesa`.
- OpenSSL 1.0 is gone from current distros. Build only the shared libraries
  (about a minute) and point `LD_LIBRARY_PATH` at them. Do not install them
  system-wide:
  ```bash
  git clone -q --depth 1 --branch OpenSSL_1_0_2u https://github.com/openssl/openssl.git ossl
  (cd ossl && ./config shared no-asm && make depend && make -j"$(nproc)")
  ```

## Trap 6: Display, Audio And Save Paths

- No display: run under `xvfb-run -a -s '-screen 0 1600x1000x24'`. Use a
  screen larger than the game window. `surface_save` captures work normally,
  so screenshot tests still produce real images.
- No audio device: PulseAudio errors in the log are expected. The runner falls
  back and keeps going. Audio playback is not verified this way.
- Save area: `working_directory` / `file_exists` resolve to
  `~/.config/<project name>/` on Linux, not `%LOCALAPPDATA%`. Stage the
  trigger file there and read the transcript from there.
- `Couldn't determine display refresh rate - assuming 60Hz` is harmless.
- The harness must **quit by itself** (`game_end()` after the transcript is
  written). Still wrap the runner in `timeout` so a hang fails instead of
  stalling the agent.

## Pass / Fail

Same priority as on Windows (`GM_CLI_HEADLESS_VERIFY.md`): the transcript wins.

```bash
grep -q COMPLETE autotest_output.txt || exit 1
! grep -q '= FAIL' autotest_output.txt || exit 1
```

Then **look at the screenshots**. A transcript cannot show a broken layout.

## Linux Is Not Windows

A green Linux run is strong evidence but not proof for a Windows release:

- Windows-only paths and scripts are not exercised.
- Font rasterization, audio and file-dialog behavior can differ.
- Have the owner run the Windows harness before merging anything platform-
  sensitive. Record which platform produced each checkpoint's evidence in the
  project's progress notes.

## Prove The Tests Can Fail

When adding a regression test with a cloud build, run it once against the old
logic (a scratch copy with the fix reverted) and confirm it reports `FAIL`. An
agent that only ever sees green has not shown the test checks anything.

## Cost Notes

- Cold setup (login, Igor, runtime, OpenSSL build): about 3–5 minutes.
- Warm rebuild + test run for a mid-size project: about 1 minute.
- Sandboxes are usually ephemeral. Keep the whole chain in a committed script
  so the next session can rebuild the setup with one command instead of
  rediscovering these traps.
