---
name: mix-engineer
description: Turn plain-English mixing requests for a running Ableton Live set into small, reversible changes, e.g. "boost the bass on the piano", "make the vocals less harsh", "more air on the hats", "turn the bass down a bit", "undo that". Use this whenever the user wants to change how a track in Ableton Live sounds (tone/EQ, level, pan) in everyday words, even if they don't say "EQ". Needs the bundled `ableton` MCP server and Live open with the AbletonMCP control surface selected.
---

# Mix engineer for Ableton Live

Your job is to turn a musician's words into **one small, safe, reversible change** in the Live set
that is running right now. Then say what you did in plain units.

You can't hear the audio. Make the standard textbook move for the words used, keep it moderate, and
make it easy to undo. Don't claim the result sounds better; that's the user's call.

## Tools (from the bundled `ableton` MCP server)

| Need | Tool |
|---|---|
| Track list with names, types and device names | `get_session_snapshot` |
| Devices on one track (index, name, `class_name`) | `get_track_info` |
| A device's parameters (`name`, `value`, `min`/`max`, `display`, `value_items`) | `get_device_parameters` |
| Find EQ Eight's browser URI | `search_browser` (`query: "EQ Eight"`, `category: "audio_effects"`) |
| Load EQ Eight onto a track | `load_instrument_or_effect` (`track_index`, `uri`) |
| Several parameter writes as **one undo step** | `batch_commands` |
| Single parameter write | `set_device_parameter`: accepts `"150 Hz"`, `"-3 dB"`, or an exact `value_items` label |
| Fader / pan | `set_track_volume` (0.0–1.0, **0.85 = 0 dB**), `set_track_pan` (−1 left … 1 right) |
| Master track | `get_master_device_parameters`, `load_device_to_master`, `set_master_device_parameter` |
| Revert | `undo` (Live's top undo entry), `redo` |

**Never call** `delete_track`, `delete_device`, `delete_*`, `replace_simpler_sample`, or `save_set`
as part of a mix request. Saving overwrites the user's `.als` file, so save only when the user
asks you to in so many words.

## Procedure

### 1. Parse the request
Pull out four things:
- **Target**: the track words ("piano", "the vox", "drums", "master").
- **Intent**: a term from [references/vocabulary.md](references/vocabulary.md), such as bass, mud,
  harsh, air, louder.
- **Direction**: "more / boost / add / bring out" is a boost. "less / cut / reduce / tame / fix /
  clean up" is a cut. Problem words (muddy, boomy, boxy, harsh, sibilant, nasal) mean a cut unless
  the user clearly asks for more.
- **Amount**: "a touch / slightly / a little / a tad" = small, plain request = medium, "a lot /
  way more / really / much" = large, an explicit number ("by 4 dB") = that number. The dB values
  are in the vocabulary file.

If the intent isn't in the vocabulary and there's no clear EQ or level meaning (e.g. "make it
groovier"), say what you *can* do instead of guessing.

### 2. Resolve the track
Call `get_session_snapshot`, then match in this order:
1. The exact track name, case-insensitive.
2. A track name that contains the word ("piano channel" → "Grand Piano", "Piano 2").
3. The synonym list in the vocabulary file ("vox" → vocal, "keys" → piano/Rhodes/EP, "kick" → BD …).
4. Device names on the track. A track called "Audio 3" whose instrument preset is "Grand Piano"
   counts as the piano.

- **Exactly one match**: go ahead.
- **None**: list the track names (with numbers) and ask which one.
- **More than one**: list the candidates and ask. Don't guess. The exception is when the user says
  "all the drums" or "every piano": then apply the move to each and say so.
- **Master or return tracks**: only when the user names them. On the master, halve the amount and
  cap boosts at +2 dB.
- **Group tracks** are valid targets. Mention that the change affects every track inside the group.

### 3. Choose the device
- **Level or pan intent**: use the mixer (step 6b), no device needed.
- **Tone intent**: call `get_track_info(track_index)` and look for EQ Eight (`class_name` `"Eq8"`,
  or a device name containing "EQ Eight").
  - **Found**: use the **last** EQ Eight in the chain, the one closest to the output, unless the
    user names another.
  - **Not found**: `search_browser("EQ Eight", "audio_effects")`, take the item named exactly
    "EQ Eight", and call `load_instrument_or_effect(track_index, uri)`. Live inserts it after the
    *selected* device, so **call `get_track_info` again** and find the new EQ Eight by comparing
    with the previous device list. Don't assume it's last.
- **Third-party EQs (VST/AU) and other devices**: don't change their parameters without asking
  first. Their parameter names aren't reliable. Adding an EQ Eight next to them is fine.

### 4. Read the EQ's current state
Call `get_device_parameters(track_index, device_index)`. Band *n* (1–8) uses these parameter names;
see [references/eq-eight.md](references/eq-eight.md):
`n Filter On A`, `n Filter Type A`, `n Frequency A`, `n Gain A`, `n Resonance A`.
Use `display` to read a value the way a human sees it, and `value_items` for the exact filter-type
labels. **Never hardcode a label.** Take the item from `value_items` that contains "Low Shelf",
"High Shelf", "Bell", "Low Cut" or "High Cut".

### 5. Pick a band, in this order
1. **Reuse**: an enabled band already doing this job (same filter family, frequency inside the
   vocabulary range). Adjust its gain or frequency instead of stacking a second band.
2. **Free band**: one whose `Filter On` is off, *or* a Bell/Shelf band sitting at 0 dB (inaudible).
   For a low-shelf or low-cut job, prefer the lowest-numbered free band. For a high-shelf or
   high-cut job, prefer the highest-numbered one. Otherwise take the lowest-numbered free band.
3. **No free band**: ask before adding a second EQ Eight.

### 6a. Write the EQ move: one `batch_commands` call, one undo step
Get the filter type, frequency and amount from the vocabulary file. That includes the
instrument-aware frequency for "bass". Then:

```json
[
  {"type": "set_device_parameter", "params": {"track_index": 3, "device_index": 1, "parameter": "2 Filter On A",   "value": 1}},
  {"type": "set_device_parameter", "params": {"track_index": 3, "device_index": 1, "parameter": "2 Filter Type A", "value": "<exact label from value_items>"}},
  {"type": "set_device_parameter", "params": {"track_index": 3, "device_index": 1, "parameter": "2 Frequency A",   "value": "150 Hz"}},
  {"type": "set_device_parameter", "params": {"track_index": 3, "device_index": 1, "parameter": "2 Gain A",        "value": "3 dB"}}
]
```

- **Reusing a band**: new gain = current gain ± amount.
- **Limits**: one request never moves a band more than **6 dB**, and a band's total stays within
  **±12 dB**. If a request would go past that, stop and suggest the alternative instead: cut the
  opposite range, or change the level.
- **Cut filters** (Low Cut / High Cut) have no gain. Set only the type and frequency.
- **Resonance (Q)**: leave it at its default for broad tonal moves. The vocabulary marks the few
  narrow moves (sibilance, nasal) where a narrower Q helps. If you set it, read `min`/`max` and
  `display` first, and confirm the readback.

### 6b. Level and pan
- **Level**: read the current `volume` from `get_track_info`. Step sizes on the 0–1 fader are:
  small ±0.03, medium ±0.05, large ±0.10. Never push it above **0.92** without asking, because 0.85
  is 0 dB and the top of the fader is +6 dB. The fader isn't linear in dB, so report it as "fader
  moved from 0.85 to 0.80", not as an exact dB value.
- **Pan**: "a bit left" = −0.25, "left" = −0.5, "hard left" = −1.0. Right is the mirror. "Center"
  = 0.

### 7. Read back and report
- Call `get_device_parameters` again and check the `display` of every parameter you touched.
  Live snaps frequencies, so "148 Hz" for a "150 Hz" request is fine.
- If any write failed, or the readback is clearly off, say so plainly and stop.
- Reply in **one or two lines**, in plain units, with the undo path:
  > **Grand Piano**: added EQ Eight; band 2 Low Shelf **+3.0 dB @ 150 Hz**. Say "undo" to revert
  > the EQ settings (twice to remove the EQ too).

  If the EQ Eight already existed, there's only one step to undo.

## Follow-ups
- **"undo" / "revert" / "put it back"**: call `undo` once for each step you made in the last
  request, and say how many.
- **"too much" / "a bit less"**: don't undo. Halve the gain you just applied on the same band.
- **"more" / "again"**: add the same amount to the same band, still within the limits.
- **"what did you change?"**: list your changes this session from memory, with track, band and
  values.

## If Live isn't reachable
On a connection error or timeout, tell the user to check three things: that Live is open, that
**AbletonMCP** is selected in Settings › Tempo & MIDI › Control Surface (Input/Output: None), and
that `uvx mcp-server-ableton-live@1.8.1 doctor --json` passes. Don't retry in a loop.

## Worked examples

**"boost the bass on the piano channel"**: the snapshot finds "Grand Piano" (track 3) with no EQ
Eight, so search for and load EQ Eight, then re-read the track to find it at device 1. Bands 1–8 are
free, so use band 1 as a Low Shelf at 150 Hz (the piano default) with +3 dB, written in one batch.
Read back "150 Hz", "3.0 dB" and report.

**"make the vocals a little less harsh"**: two tracks match, "Lead Vox" and "Vox Dbl", so ask which
one, or both. Once the user picks Lead Vox, which already has an EQ Eight, take a free band as a
Bell at 3.5 kHz with −1.5 dB (small).

**"turn the bass down a lot"**: this is a level request. "Bass" matches a *track* name, so this is
the Bass track's fader, not an EQ move. Move the fader from 0.85 to 0.75 and report it. If no track
is called bass, this phrase instead means cutting bass on whatever track the user names.
