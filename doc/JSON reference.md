# Mini-Moon JSON Preset Reference

This document defines the structure and field-level specification for Mini-Moon presets written in JSON format. It is intended for developers or advanced users who wish to generate or manipulate presets programmatically.

---

## General Structure

```json
{
  "modules": {
    "oscillator": [...],
    "lfo": [...],
    "filter": [...],
    "effect": [...],
    "envelope": { ... },
    "dynamicEnvelope": [...],
    "master": { ... }
  }
}
```

Each key under `modules` defines a specific synthesis component. Sections may be omitted to disable a given module type.

---

## Oscillator

**Type**: Array of objects

```json
{
  "name": "OSC-1",
  "type": "OscillatorType.SquarePWM",
  "gain": 1.0,
  "pan": 0.0,
  "transpose": 0,
  "detune": 0,
  "phaseOffset": 0.0,
  "randomPhase": false,
  "pulseWidth": 0.5,
  "pulseWidthMod": 0.0,
  "fmAmount": 0.0
}
```

### Fields

* `name` *(string)* – Unique name of the oscillator.
* `type` *(enum)* – One of `OscillatorType`.
* `gain`, `pan`, `transpose`, `detune`, `phaseOffset`, `randomPhase` – Voice shaping parameters.
* `pulseWidth` *(float 0.01–0.99)* – Duty cycle for PWM-type oscillators.
* `pulseWidthMod` *(float 0.0–1.0)* – Sensitivity to LFO modulation.
* `fmAmount` *(float)* – Amount of frequency modulation from another oscillator.

---

## LFO

**Type**: Array of objects

```json
{
  "name": "LFO-1",
  "waveform": "LFOWaveform.Sine",
  "frequency": 1.0,
  "depth": 1.0,
  "syncToTempo": false,
  "division": 1.0,
  "assignment": "LFOAssignment.OscPWM",
  "target": "PWM-1",
  "callback": {
    "interval": 8,
    "loop": true,
    "func": "(Lua function reference)"
  }
}
```

### Fields

* `name` *(string)* – Unique identifier.
* `waveform` *(enum)* – One of `LFOWaveform`.
* `frequency`, `depth` *(float)* – Rate and intensity of modulation.
* `syncToTempo` *(bool)* – If true, ignores frequency and uses division.
* `division` *(float)* – Rhythmic subdivision (e.g. 0.5 = 8th note).
* `assignment` *(enum)* – Built-in modulation routing.
* `target` *(string)* – Optional name of module to affect.
* `callback` *(object)* – Optional table for Lua-based modulation.

---

## Filter

**Type**: Array of objects

```json
{
  "name": "LPF",
  "type": "FilterType.LowPass",
  "cutoff": 1200.0,
  "resonance": 0.5,
  "gain": 0.0,
  "slope": 0,
  "bypass": false
}
```

### Fields

* `type` *(enum)* – One of `FilterType`.
* `cutoff`, `resonance`, `gain`, `slope` *(float)* – Filter shaping.
* `bypass` *(bool)* – Disable this filter.

---

## Effect

**Type**: Array of objects

Each effect has different parameters depending on its `type`. All share:

* `name` *(string)*
* `type` *(enum: EffectType)*
* `mix` *(float)* – Wet/dry balance
* `bypass` *(bool)* – Disable effect

### Common Effect Types

| Type         | Parameters                                                             |
| ------------ | ---------------------------------------------------------------------- |
| `Reverb`     | `size`, `damping`                                                      |
| `Delay`      | `time`, `feedback`                                                     |
| `Chorus`     | `rate`, `depth`                                                        |
| `Flanger`    | `rate`, `depth`, `delay`                                               |
| `Rotary`     | `rate`, `depth`                                                        |
| `Distortion` | `mode` (enum: DistortionType), `gain`, `bias`, `bitDepth`, `reduction` |
| `Compressor` | `threshold`, `ratio`, `attack`, `release`, `makeup`, `knee`            |
| `Limiter`    | `threshold`, `attack`, `release`                                       |
| `NoiseGate`  | `threshold`, `attack`, `release`                                       |

---

## Envelope

**Type**: Single object

```json
{
  "attack": 0.01,
  "decay": 0.2,
  "sustain": 0.8,
  "release": 0.4
}
```

Standard ADSR settings. Values are time (s) or level (0.0–1.0).

---

## DynamicEnvelope

**Type**: Array of objects

```json
{
  "points": [
    { "t": 0.0, "v": 0.0 },
    { "t": 0.2, "v": 1.0, "mode": "exp" },
    { "t": 1.0, "v": 0.0 }
  ],
  "callback": {
    "interval": 4,
    "loop": true,
    "func": "(Lua function reference)"
  }
}
```

### Fields

* `points` *(array of objects)*:

  * `t` *(float)* – Time in seconds
  * `v` *(float)* – Value at that point
  * `mode` *(string)* – Interpolation mode between points
  * `cp1`, `cp2` *(float, optional)* – Control points (e.g. for Bezier)
* `callback` *(object)* – Identical to LFO-style callback

### Supported Modes

* `linear`
* `exp`
* `log`
* `sine`
* `step`
* `hold`

---

## Master

**Type**: Single object

```json
{
  "gain": 1.0,
  "tempo": 120.0,
  "pan": 0.0,
  "bpm": 120.0,
  "portamento": 0.0,
  "transpose": 0.0
}
```

### Fields

* `gain` *(float)* – Overall output gain
* `tempo` / `bpm` *(float)* – Global tempo (BPM)
* `pan` *(float)* – Global stereo position
* `portamento` *(float)* – Portamento time in seconds
* `transpose` *(float)* – Global pitch shift in semitones

---

## End of Reference
