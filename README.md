# ableton-mix: plain-English mixing for Ableton Live

> "Boost the bass on the piano channel."
> → **Grand Piano**: added EQ Eight; band 1 Low Shelf **+3.0 dB @ 150 Hz**. Say "undo" to revert.

A Claude Code plugin that turns everyday mixing language into small, reversible changes in the
Live set that's currently open. It's the "mix engineer" layer on top of an existing, reviewed
open-source Ableton MCP server.

## How it fits together

```
You ──"more air on the hats"──▶ Claude ──skill: mix-engineer──▶ MCP server (uvx, pinned)
                                                                  │  localhost:9877
                                                                  ▼
                                                   Ableton Live ◀─ AbletonMCP Remote Script
```

| Piece | What it is | Who wrote it |
|---|---|---|
| `plugins/ableton-mix/skills/mix-engineer/` | The procedure, the vocabulary → EQ table, and EQ Eight notes | this repo |
| `plugins/ableton-mix/commands/mix.md` | `/mix <request>` shortcut | this repo |
| `plugins/ableton-mix/.mcp.json` | Starts [`mcp-server-ableton-live`](https://github.com/wstierhout/ableton-live-mcp) **1.8.1** via `uvx` | MIT, wstierhout |
| AbletonMCP Remote Script | Runs inside Live and talks to the server over a local socket | ships with the server |

The plugin itself is only Markdown and JSON. It contains no executable code.

## Requirements (on the computer that runs Ableton)

Written for **Windows 10/11**. On macOS the steps are the same; only the commands in the
[macOS note](#macos) differ.

- Ableton Live **12** (built and reviewed for 12.3+ Suite; 11 should work too)
- [Claude Code](https://code.claude.com/docs/en/setup) for Windows
- [uv](https://docs.astral.sh/uv/), which fetches Python 3.10+ by itself, so you don't need to install Python
- [Git for Windows](https://git-scm.com/downloads/win), to clone this repo. Claude Code also uses it
  for its Bash tool.

## Install on Windows (about 10 minutes)

Run all commands in **PowerShell** (Start menu → "PowerShell"; the prompt starts with `PS C:\`).
After installing a tool, **open a new PowerShell window** so it's on your PATH.

1. **Install the tools** (skip any you already have):
   ```powershell
   winget install --id=astral-sh.uv -e
   winget install --id=Git.Git -e
   irm https://claude.ai/install.ps1 | iex
   ```
   Open a new PowerShell window and check: `uvx --version`, `git --version`, `claude --version`.

2. **Get this repo.** Until the PR is merged, the plugin lives on the `feat/mix-engineer-plugin`
   branch:
   ```powershell
   cd $HOME\Documents
   git clone -b feat/mix-engineer-plugin https://github.com/gal-dn/ableton-mix-plugin
   ```
   Git will open a browser window to sign in to GitHub the first time, because the repo is private.

3. **Install the Live-side Remote Script** (once, and again after any server upgrade):
   ```powershell
   uvx mcp-server-ableton-live@1.8.1 install
   ```
   It copies the script to `Documents\Ableton\User Library\Remote Scripts`. If you get
   *"User Library not found"*, see [Troubleshooting](#troubleshooting-windows).

4. **Turn it on in Live.** Fully quit and reopen Live. Open **Options › Settings** (`Ctrl+,`) and
   go to the **Tempo & MIDI** tab. In an unused **Control Surface** row choose **AbletonMCP**, and
   set that row's **Input** and **Output** to **None**.

5. **Check it**, with a Live set open:
   ```powershell
   uvx mcp-server-ableton-live@1.8.1 doctor --json
   ```

6. **Start Claude Code with the plugin.** Either:
   - **try it for one session:**
     ```powershell
     claude --plugin-dir "$HOME\Documents\ableton-mix-plugin\plugins\ableton-mix"
     ```
   - **or install it** so it loads every time:
     ```powershell
     claude plugin marketplace add "$HOME\Documents\ableton-mix-plugin"
     claude plugin install ableton-mix@ableton-mix-plugin
     ```
     After the PR is merged you can use `claude plugin marketplace add gal-dn/ableton-mix-plugin`
     instead of the local path.

   Inside Claude Code, `/mcp` should list **ableton** as connected.

### Troubleshooting (Windows)

| Symptom | Fix |
|---|---|
| `uvx`, `git` or `claude` "is not recognized" | Open a **new** PowerShell window. If it still fails, sign out of Windows and back in, so PATH refreshes. |
| `install` says *"User Library not found"* | Your Documents folder is probably synced by **OneDrive**. In Live, open **Options › Settings › Library** and copy the *User Library* path, then run `uvx mcp-server-ableton-live@1.8.1 install --user-library "<that path>"`. Pass the same `--user-library` to `doctor`. |
| **AbletonMCP** isn't in the Control Surface list | Quit Live completely (check the system tray), reopen it, and look again. Re-run `install` if needed. |
| `doctor` can't connect | Live must be open with the AbletonMCP row selected. The bridge only talks over `127.0.0.1:9877`, so it doesn't need a Windows Firewall exception. If another app uses port 9877, see the [server README](https://github.com/wstierhout/ableton-live-mcp#readme) for changing the port. |
| `/mcp` shows **ableton** failed | Run `uvx mcp-server-ableton-live@1.8.1 --version` in PowerShell. It must print `1.8.1` there before Claude Code can start it. |

### macOS

Same steps, with these differences: install the tools with `brew install uv git` plus
`curl -fsSL https://claude.ai/install.sh | bash`. The default User Library is
`~/Music/Ableton/User Library`. Live's settings are under **Live › Settings** (`Cmd+,`), and paths
use `/` (e.g. `claude --plugin-dir ~/Documents/ableton-mix-plugin/plugins/ableton-mix`).

## Use it

```
/mix boost the bass on the piano
/mix make the vocals a little less harsh
/mix more air on the hats
/mix turn the bass down a bit
undo
```

Plain sentences work too: the skill triggers on mixing requests without the slash command.
See [docs/TESTING.md](docs/TESTING.md) for the acceptance checklist.

**Coming next** ([roadmap](docs/ROADMAP.md)): a `producer` skill for style-level requests such as
"mix it like a Stevie Wonder song" or "make it a dark 80s mix", then matching your mix against
reference tracks you supply.

## What it will and won't do

**Will**
- Find the track by name, synonym ("vox", "keys") or instrument preset. If it's ambiguous, it asks.
- Reuse an existing EQ Eight (and an existing band doing the same job) before adding anything.
- Make moderate moves: 1.5 / 3 / 6 dB, at most 6 dB per request and ±12 dB per band.
- Write each change as **one undo step**, read it back from Live, and report it in plain units.

**Won't**
- Delete tracks, devices or clips, replace samples, or save your set.
- Change third-party plugin (VST/AU) parameters without asking first.
- Pretend to hear. Moves are textbook starting points, so trust your ears.
- v1 doesn't cover width, compression, reverb, saturation, automation or sidechain.

## Security notes

- The reviewed server, `mcp-server-ableton-live` **1.8.1**, binds to `127.0.0.1:9877` and has no
  authentication. Any process on your computer can drive Live while the control surface is on.
  Don't expose that port to a network.
- Review findings for 1.8.1: no telemetry or outbound HTTP calls. The only subprocess is its own
  audio decoder (`_audio_decode.py`), used by the audio-file analysis tool.
- The version is **pinned**. To upgrade: review the new release, update the version in
  `.mcp.json` and in the commands above, and re-run `install` with the new version.

## Known limitations

- Fader moves are reported as fader positions (0.85 = 0 dB), not exact dB, because Live's fader
  isn't linear.
- A newly loaded EQ Eight lands after the *selected* device. The skill re-reads the chain to find
  it, but the position may not be last.
- Live's Remote Script API is unofficial and can change between Live versions. Re-run `doctor`
  after updating Live.
