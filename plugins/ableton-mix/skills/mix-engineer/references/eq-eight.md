# EQ Eight: what the plugin relies on

This is Ableton's 8-band EQ, built into every edition of Live. Its `class_name` in the Live Object
Model is `Eq8`.

## Per-band parameters (band n = 1…8)

| Parameter name | What it does | How to write it |
|---|---|---|
| `n Filter On A` | Band on/off | native `1` (on) / `0` (off) |
| `n Filter Type A` | Filter shape | the exact label from this parameter's `value_items` (e.g. the item containing "Low Shelf", "Bell", "High Shelf", "Low Cut", "High Cut") |
| `n Frequency A` | Centre / corner frequency | display text with units: `"150 Hz"`, `"3.5 kHz"` |
| `n Gain A` | Boost/cut, about ±15 dB | display text: `"3 dB"`, `"-1.5 dB"` (no effect on cut filters) |
| `n Resonance A` | Q / bandwidth | leave at default for broad moves. To change it, read `min`/`max`/`display` first |

**The `A` suffix:** EQ Eight's *Mode* can be Stereo, L/R or M/S. In Stereo, only the `A` set
applies. In L/R or M/S, `A` is Left/Mid and a parallel `B` set exists. The plugin only edits `A`.
If the device's Mode isn't Stereo, tell the user which side you changed.

## Device-wide parameters worth knowing

- `Device On`: don't touch it.
- `Output Gain`: don't use it for tonal requests. It's a level change.
- `Adaptive Q`, `Scale`: leave them alone.

## Reading the state

`get_device_parameters` returns, for each parameter: `index`, `name`, `value` (native), `min`,
`max`, `is_quantized`, `display` (what Live shows, e.g. "150 Hz"), and for quantized parameters
`value_items` (the allowed labels).

A band is **free** if `Filter On` is off, *or* it's a Bell/Shelf with `Gain` at 0 dB. A cut filter
that is on is never free: it's audible at any setting.

## Why labels aren't hardcoded

Filter-type label text can differ between Live versions (e.g. how the Low Cut slope is written).
Always pick from the live `value_items` by substring.
