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

- Ableton Live **12** (built and reviewed for 12.3+ Suite; 11 should work too)
- [Claude Code](https://claude.com/claude-code) CLI
- [uv](https://docs.astral.sh/uv/) (`brew install uv`); it fetches Python 3.10+ if needed

## Install (about 5 minutes)

1. **Get this repo** onto the Ableton computer: clone it, or unpack the tarball.
2. **Install the Live-side Remote Script** (one time, and again after any server upgrade):
   ```bash
   uvx mcp-server-ableton-live@1.8.1 install
   ```
3. **Turn it on in Live.** Fully quit and reopen Live. Open **Settings › Tempo & MIDI**, and in an
   unused **Control Surface** row choose **AbletonMCP**. Set that row's **Input** and **Output**
   to **None**.
4. **Check it**, with a Live set open:
   ```bash
   uvx mcp-server-ableton-live@1.8.1 doctor --json
   ```
5. **Load the plugin in Claude Code.** Either:
   - **try it for one session:**
     ```bash
     claude --plugin-dir /path/to/ableton-mix-plugin/plugins/ableton-mix
     ```
   - **or install it:**
     ```bash
     claude plugin marketplace add /path/to/ableton-mix-plugin
     claude plugin install ableton-mix@ableton-mix-plugin
     ```
     Once this repo is on GitHub, use `claude plugin marketplace add <owner>/ableton-mix-plugin`.

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
