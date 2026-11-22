# Acceptance checklist (run on the Ableton computer)

Takes about 20 minutes. Do it on a **copy** of a set, or on the test set below. Nothing here
saves your set, but it's still a good habit.

## 0. Setup

- [ ] `uvx mcp-server-ableton-live@1.8.1 doctor --json` passes with a Live set open.
- [ ] Claude Code is started with the plugin (on Windows:
      `claude --plugin-dir "$HOME\Documents\ableton-mix-plugin\plugins\ableton-mix"`).
      `/mix` appears in the slash-command list, and `/mcp` shows **ableton** as connected.

## 1. Test set

Create a new Live set with these tracks (any clips, ideally looping so you can listen):

| # | Track name | Contents |
|---|---|---|
| 1 | Kick | Drum Rack or audio kick |
| 2 | Bass | any bass instrument |
| 3 | Grand Piano | Grand Piano preset, **no effects** |
| 4 | Lead Vox | audio or any instrument, with **one EQ Eight** already on it |
| 5 | Vox Dbl | anything |
| 6 | Pad | any pad, with a **third-party EQ plugin** if you have one |

## 2. Core scenarios

| # | Say | Expected |
|---|---|---|
| 1 | `/mix boost the bass on the piano channel` | EQ Eight added to **Grand Piano**; one band = Low Shelf, about 150 Hz, **+3.0 dB**; one-line report with the undo path |
| 2 | `a bit less` | Same band drops to about +1.5 dB (no new band, no undo) |
| 3 | `undo` | Band returns to its previous value; Claude says how many steps it undid |
| 4 | `/mix make the vocals less harsh` | Asks: Lead Vox, Vox Dbl, or both? |
| 5 | (answer) `Lead Vox` | Uses the **existing** EQ Eight (no second EQ); Bell about 3.5 kHz, **−3 dB** |
| 6 | `/mix a lot more air on the lead vox` | High Shelf about 12 kHz, **+6 dB** (the per-request cap) on Lead Vox |
| 7 | `/mix even more air` (repeat 2–3 times) | Stops at ±12 dB total and suggests an alternative instead of going further |
| 8 | `/mix more thump on the kick` | Bell about 60 Hz (not a shelf), +3 dB |
| 9 | `/mix turn the bass down a bit` | **Bass** track fader −0.03 (e.g. 0.85 → 0.82), not an EQ move |
| 10 | `/mix brighten the pad` | Adds EQ Eight next to the third-party EQ; **asks first** if it wanted to change the plugin |
| 11 | `/mix boost the bass on the trumpet` | No match: lists tracks and asks |
| 12 | `/mix make it groovier` | Explains it can't do that, and offers what it can |

## 3. Safety checks

- [ ] No track, device or clip was deleted during the run.
- [ ] The set's title bar still shows unsaved changes (it never saved for you).
- [ ] Undo in Live's own Edit menu reverts each Claude change in one step (two steps for "added
      EQ Eight + settings").

## 4. Ear check (subjective, the real test)

For scenarios 1, 5 and 8, A/B with the EQ Eight on and off. Is the move in the right direction and
a sensible size? Note anything that sounds wrong. Those notes tune `references/vocabulary.md`.

## Reporting back

For anything that fails, note the scenario number, what Claude said, and what Live shows (the EQ
Eight band values). Copy the `doctor --json` output if the connection fails.
