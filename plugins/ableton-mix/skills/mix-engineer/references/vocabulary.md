# Mixing vocabulary → EQ Eight moves

These are textbook starting points, not rules. The plugin can't hear the mix, so every move here
is moderate and reversible.

## Amounts

| Wording | Boost / cut |
|---|---|
| "a touch", "slightly", "a little", "a tad", "a hair" | **1.5 dB** |
| no qualifier | **3 dB** |
| "a lot", "way more", "really", "much", "heaps" | **6 dB** (the per-request maximum) |
| explicit number ("by 4 dB") | that number, still capped at 6 dB per request |

## Tone terms

**Boost** is the move for "more X". **Cut** is the move for "less X" (or "fix X" for problem words).
"—" means the term only makes sense one way: if the user asks for the other direction, ask what
they mean.

| Term (and synonyms) | Range | Boost | Cut |
|---|---|---|---|
| **sub**, sub-bass, deep end | 20–60 Hz | Low Shelf @ 60 Hz | Low Shelf @ 60 Hz |
| **rumble**, "clean up the low end" | < 40 Hz (up to 100 Hz on non-bass parts) | — | Low Cut @ 35 Hz (instruments with no real low end, e.g. hats, vocals, guitars: @ 80–100 Hz). Use the gentlest Low Cut in `value_items` unless the user says "steep" |
| **bass**, low end, bottom, weight, thump, boom (when wanted) | 60–250 Hz | Low Shelf @ *instrument default* (see below) | Low Shelf @ *instrument default* |
| **warmth**, warm, fat | 150–400 Hz | Bell @ 250 Hz | Bell @ 250 Hz |
| **body**, fullness, thickness, "less thin" | 150–300 Hz | Bell @ 200 Hz | Bell @ 200 Hz |
| **mud**, muddy, cloudy, boomy | 200–500 Hz | — | Bell @ 300 Hz |
| **boxy**, honky, cardboard | 400–900 Hz | — | Bell @ 600 Hz |
| **nasal** | 800 Hz–1.5 kHz | — | Bell @ 1 kHz, narrower Q (about 2) |
| **presence**, forward, clarity, "cut through", definition | 2–5 kHz | Bell @ 3 kHz | Bell @ 3 kHz |
| **harsh**, harshness, piercing, shrill, brittle | 2.5–5 kHz | — | Bell @ 3.5 kHz |
| **sibilance**, esses, "ssss" | 5–9 kHz | — | Bell @ 7 kHz, narrower Q (about 3). Mention that a de-esser does this better |
| **bright**, brightness, sparkle, crisp, "less dull", "less muffled" | 5–10 kHz | High Shelf @ 6 kHz | High Shelf @ 6 kHz |
| **dark**, darker, "less bright", "less tinny" | 5–10 kHz | — (means a cut) | High Shelf @ 6 kHz |
| **air**, airy, open, "more top" | 10–16 kHz | High Shelf @ 12 kHz | High Shelf @ 12 kHz |
| **hiss**, "too much top" | > 10 kHz | — | High Cut @ 12 kHz (gentlest High Cut in `value_items`) |

### Instrument defaults for "bass" (Low Shelf frequency)

| Track looks like | "bass" shelf | Note |
|---|---|---|
| kick, BD, 808 | **Bell @ 60 Hz** (not a shelf) | thump lives in a narrow region |
| bass, sub, synth bass, bass guitar | 80 Hz | |
| drums / drum bus / group | 80 Hz | |
| piano, keys, Rhodes, EP, organ | 150 Hz | |
| guitar, acoustic | 150 Hz | |
| pad, synth, strings, choir | 150 Hz | |
| vocal, vox, voice | **Bell @ 200 Hz** ("body") | a vocal low shelf usually just adds mud |
| master | 80 Hz | halve the amount; boosts capped at +2 dB |
| anything else | 120 Hz | |

## Level and pan terms

| Term | Move |
|---|---|
| louder, up, "turn up", "bring up", "push" | fader +0.05 (small +0.03, large +0.10), never above 0.92 without asking |
| quieter, softer, down, "turn down", "pull back", "tuck in" | fader −0.05 (small −0.03, large −0.10) |
| "a bit left/right" | pan ∓/± 0.25 |
| "left/right" | pan ∓/± 0.5 |
| "hard left/right" | pan ∓/± 1.0 |
| "center", "middle" | pan 0 |

## Not covered in v1 (say so instead of improvising)

Stereo width, compression or "punch", reverb or "space", saturation or "grit", automation over
time, sidechain or "pumping". Offer what the plugin *can* do, e.g. "I can add presence at 3 kHz,
which often reads as more punch".

## Track-name synonyms

| Words the user says | Track names to look for |
|---|---|
| vox, vocal, voice, singer, lead | vox, vocal, voc, lead, ld, voice |
| keys, piano | piano, pno, keys, rhodes, ep, wurli, organ |
| kick | kick, kck, bd, 808 |
| snare | snare, snr, sd, clap |
| hats, hi-hats | hat, hh, hihat, hi-hat, cymbal |
| drums, kit, beat | drum, drums, kit, beat, perc (group tracks too) |
| bass | bass, sub, bs, 808 (only when no kick-like 808 exists) |
| guitar | gtr, guitar, acoustic, ac gtr |
| synth, lead synth | synth, lead, pluck, arp |
| pad | pad, strings, str, choir |
