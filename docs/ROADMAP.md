# Roadmap

| Version | Theme | Status |
|---|---|---|
| **v0.1** | `mix-engineer` skill: plain-English EQ, level and pan on one track | ✅ built, awaiting a test on Live (Windows) |
| **v0.2** | `producer` skill: style- and vibe-level mixing ("mix it like a dark 80s song") | 📝 planned |
| **v0.3** | Reference matching: compare your mix against reference tracks you supply | 📝 planned |

---

## v0.2: `producer` skill (master producer / DJ)

### Goal
Let the user talk like a producer, not an engineer:

> "Mix this song like a Stevie Wonder song."
> "Make it a dark 80s mix."
> "Make the drums punchy and mellow out the keys."

The producer skill decides **what** the mix needs and **why**. It writes a short *mix brief* and
hands each move to `mix-engineer`, which already knows **how** to change devices in Live safely.

### How it works

1. **Understand the ask.** Classify the request:
   - a *style or era* ("dark 80s", "70s soul")
   - an *artist-as-style* ("like a Stevie Wonder song")
   - a *genre* ("boom bap", "deep house")
   - a *vibe word* ("punchy", "mellow", "lush", "gritty")
2. **Get the knowledge**, in this order:
   1. **Bundled style library** (`skills/producer/references/styles/*.md`): curated profiles,
      one per style (see the format below).
   2. **Descriptor vocabulary** (`references/descriptors.md`): vibe words mapped to multi-device
      moves, e.g. *punchy* = slower compressor attack, transient emphasis and less low-mid
      clutter; *mellow* = softer highs, gentler transients and a touch of warmth.
   3. **Web research** for anything not bundled: search production write-ups, interviews and
      gear lists. Summarize them into the same profile format **with cited sources**, show the
      user, and offer to save the result as a custom style.
3. **Read the session.** Take a snapshot of the tracks and classify each one by role (kick, snare,
   drums bus, bass, keys, guitar, pads, lead, vocals, FX) from its name and instruments. Ask
   about anything unclear.
4. **Write the mix brief.** List 5–12 concrete moves grouped by role. Each move names the device,
   parameter and value, plus a one-line "why" in producer terms:

   | Role | Move | Why |
   |---|---|---|
   | Snare | Send −12 dB to a return with Reverb → Gate (short, bright hall) | the 80s gated snare |
   | Bass | Chorus (via a rack or Chorus-Ensemble), mix 25 % | that cold, wide 80s bass |
   | Pads | Utility width 130 %, High Shelf −2 dB @ 8 kHz | wide but dark |

5. **Confirm, then apply.** Multi-track changes always need a yes ("apply all", "only drums",
   "skip the reverb"). Load any devices first, then write every parameter in **one batch, so
   one undo reverts the whole vibe**. Read the values back and report.
6. **Iterate.** Follow-ups like "less", "darker" or "more like the chorus of …" adjust the same
   moves instead of stacking new ones.

### Style profile format (one file per style)
```
name / aliases           e.g. "70s soul-funk", "Stevie Wonder-era", "Motown-adjacent"
era & context            years, typical gear, recording approach
tonal balance            what's forward / recessed (lows, low-mids, presence, air)
dynamics                 punchy vs. squashed; typical compression character
space                    dry vs. roomy; reverb types and lengths; delay use
stereo image             mono-ish vs. wide; what sits in the centre
colour                   tape/console saturation, chorus, filters, lo-fi
per-role moves           drums / bass / keys / guitars / pads / vocals / master
signature moves          the 2–3 things that make it sound like the style
avoid                    moves that break the style
tempo & feel (info only) typical BPM / swing; never changed without asking
sources                  links used to build or verify the profile
```

**Starter library (12 profiles):**
- 70s soul-funk (Stevie Wonder-era: Clavinet and Rhodes forward, dry tight drums, round upfront
  bass, warm tape colour)
- dark 80s / darkwave / post-punk (gated snare, chorus bass, cold wide synths, big plates and
  halls)
- 80s pop
- 90s boom-bap hip hop
- lo-fi hip hop
- modern trap
- deep house
- techno
- indie rock
- jazz trio
- ambient
- disco

Each profile is researched and cited when it's written.

### Wider device coverage (an extension to `mix-engineer`)
v0.1 only uses EQ Eight, the faders and pan. The producer skill needs `mix-engineer` to cover these
devices too. The server already has curated guidance for all of them through `describe_device`:

| Device | Used for |
|---|---|
| Compressor / Glue Compressor | punch, glue, smoothness |
| Saturator | warmth, grit, "tape" |
| Utility | width, mono bass, gain staging |
| Reverb / Delay on **return tracks** plus track sends | space, depth, era-specific ambience |
| Auto Filter | dark, lo-fi, movement |
| Limiter (master only, with explicit OK) | final loudness |

### Guardrails (in addition to v0.1's)
- **Confirm before any multi-track or master-bus change.** Show the brief first.
- **Never change tempo, key or arrangement.** Style profiles may *mention* typical BPM, as
  information only.
- **No auto-downloading of audio.** The skill won't fetch commercial recordings from the web,
  because of copyright and site terms. Web research is text only: articles, interviews, gear
  lists. Reference audio comes from **files the user supplies** (see v0.3).
- **Style, not copying.** Artist names are shorthand for a production aesthetic. The skill
  recreates mix traits, never parts of the recording itself.
- Cite sources for any researched profile, and say when a profile is a best guess.

### Acceptance scenarios (added to `docs/TESTING.md` when built)
1. "Mix this like a Stevie Wonder song" → bundled 70s soul-funk profile, a brief of 6–10 moves,
   applied after a yes, one undo reverts everything.
2. "Make it a dark 80s mix" → gated snare reverb via a return track, chorus on the bass, darker
   pads, wider synths.
3. "Make the drums punchy" → Compressor on the drums with a slower attack, plus small EQ moves.
   No other tracks change.
4. "Mellow out the keys" → softer highs and gentler dynamics on the keys only.
5. "Mix it like 2000s Swedish melodic death metal" (not bundled) → web research, a profile with
   sources, and an offer to save it before applying.
6. "Do the 80s thing but less reverb" → adjusts the same sends, nothing new is stacked.

**Effort:** about 2–3 days. Writing and citing the 12 profiles is the biggest part.

---

## v0.3: Reference matching

### Goal
"Make my mix sit like *this* track", where *this* is an audio file the user owns.

### Plan
- **Gap:** the server's `analyze_audio_file` gives peak, RMS and crest factor, but no tonal
  balance, LUFS or stereo width.
- **Add a small companion MCP server** in this repo (`analyzer/`, Python run with `uv`, started
  from the plugin's `.mcp.json` via `${CLAUDE_PLUGIN_ROOT}`). It has two tools:
  - `analyze_reference(path)` → integrated LUFS, loudness range, crest factor, octave-band tonal
    balance, stereo correlation/width, low-end mono-ness.
  - `compare_to_reference(mix_path, reference_path)` → per-metric differences in plain words
    ("your low-mids are 3 dB heavier, your mix is 4 LU quieter, you're narrower above 2 kHz").
- **The loop:** the user exports a bounce (Live: *File › Export Audio/Video*). The producer skill
  compares it with the reference, proposes moves through the normal brief and confirm flow,
  applies them, and the user re-bounces to check.
- **Files are local only.** Nothing is uploaded. Windows paths with spaces and OneDrive-synced
  folders are supported.

**Effort:** about 2–3 days, including tests with fixture audio files.
