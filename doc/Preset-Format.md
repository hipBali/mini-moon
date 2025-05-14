# Preset Format and Parameter Specification

This document describes the structure and content of Mini-Moon presets, supporting both `.lua` and `.json` formats.

---

## Preset Structure Overview

Mini-Moon supports three preset modes:

1. **Pure Lua preset** – scripting and module configuration in one file
2. **Pure JSON preset** – configuration only, no scripting support
3. **Hybrid preset** – JSON for module configuration, Lua file for scripting callbacks

The core of every preset is a table (or JSON object) called `modules`, which contains keys like `oscillator`, `lfo`, `filter`, `envelope`, `effect`, and `master`.

> **Important**:
>
> * If a module is not defined, it is considered **bypassed**.
> * `envelope` and `master` sections are always **present and active**, even if not explicitly defined.
> * Only the **oscillator** section must be explicitly defined.
> * You can define **any number** of oscillators, LFOs, filters, and effects.
> * **Order matters** only in the `filter` and `effect` sections (serial signal path).

---

## Full Example

### JSON

```json
{
  "modules": {
    "oscillator": [
      {
        "name": "OSC-1",
        "type": "OscillatorType.Sine"
      }
    ],
    "lfo": [
      {
        "name": "LFO-1",
        "waveform": "LFOWaveform.Sine",
        "assignment": "LFOAssignment.OscGain",
        "bypass": true
      }
    ],
    "filter": [
      {
        "name": "LPF",
        "type": "FilterType.LowPass",
        "bypass": true
      },
      {
        "name": "HPF",
        "type": "FilterType.HighPass",
        "bypass": true
      },
      {
        "name": "BPF",
        "type": "FilterType.BandPass",
        "bypass": true
      },
      {
        "name": "MLP",
        "type": "FilterType.MoogLowPass",
        "bypass": true
      },
      {
        "name": "NOF",
        "type": "FilterType.Notch",
        "bypass": true
      }
    ],
    "envelope": {
      "attack": 0.05,
      "decay": 0.05,
      "sustain": 0.05,
      "release": 0.05
    },
    "effect": [
      {
        "name": "Distortion",
        "type": "EffectType.Distortion",
        "mode": "DistortionType.Overdrive",
        "gain": 5.0,
        "mix": 0.5,
        "bias": 0.0,
        "bitDepth": 8,
        "reduction": 4,
        "bypass": true
      },
      {
        "name": "Chorus",
        "type": "EffectType.Chorus",
        "rate": 0.3,
        "depth": 0.4,
        "mix": 0.5,
        "bypass": true
      },
      {
        "name": "Delay",
        "type": "EffectType.Delay",
        "time": 0.3,
        "feedback": 0.4,
        "mix": 0.5,
        "bypass": true
      },
      {
        "name": "Reverb",
        "type": "EffectType.Reverb",
        "size": 0.5,
        "damping": 0.5,
        "mix": 0.5,
        "bypass": true
      },
      {
        "name": "Flanger",
        "type": "EffectType.Flanger",
        "rate": 0.2,
        "depth": 0.5,
        "delay": 2.0,
        "mix": 0.5,
        "bypass": true
      },
      {
        "name": "Rotary",
        "type": "EffectType.Rotary",
        "rate": 0.3,
        "depth": 0.6,
        "mix": 0.5,
        "bypass": true
      },
      {
        "name": "Compressor",
        "type": "EffectType.Compressor",
        "threshold": -18.0,
        "ratio": 4.0,
        "attack": 0.01,
        "release": 0.3,
        "makeup": 0.0,
        "knee": 0.0,
        "bypass": true
      },
      {
        "name": "Limiter",
        "type": "EffectType.Limiter",
        "threshold": -1.0,
        "attack": 0.001,
        "release": 0.1,
        "bypass": true
      },
      {
        "name": "NoiseGate",
        "type": "EffectType.NoiseGate",
        "threshold": -40.0,
        "attack": 0.01,
        "release": 0.1,
        "bypass": true
      }
    ],
    "master": {
      "gain": 1.0,
      "tempo": 90
    }
  }
}

```

### LUA 

```lua
modules = {
  oscillator = {
    { name = "OSC-1", type = OscillatorType.Sine },
  },
  lfo = {
    { name = "LFO-1", waveform = LFOWaveform.Sine, assignment = LFOAssignment.OscGain, bypass = true },
  },
  filter = {
    { name = "LPF", type = FilterType.LowPass,  bypass = true },
    { name = "HPF", type = FilterType.HighPass, bypass = true },
    { name = "BPF", type = FilterType.BandPass, bypass = true },
    { name = "MLP", type = FilterType.MoogLowPass, bypass = true },
    { name = "NOF", type = FilterType.Notch, bypass = true }
  },
  envelope = {
    attack = 0.05,
    decay = 0.05,
    sustain = 0.05,
    release = 0.05
  },
  effect = {
    { name = "Distortion", type = EffectType.Distortion, mode = DistortionType.Overdrive, gain = 5.0, mix = 0.5, bias = 0.0, bitDepth = 8, reduction = 4, bypass = true },
    { name = "Chorus",     type = EffectType.Chorus, rate = 0.3, depth = 0.4, mix = 0.5, bypass = true },
    { name = "Delay",      type = EffectType.Delay, time = 0.3, feedback = 0.4, mix = 0.5, bypass = true },
    { name = "Reverb",     type = EffectType.Reverb, size = 0.5, damping = 0.5, mix = 0.5, bypass = true },
    { name = "Flanger",    type = EffectType.Flanger, rate = 0.2, depth = 0.5, delay = 2.0, mix = 0.5, bypass = true },
    { name = "Rotary",     type = EffectType.Rotary, rate = 0.3, depth = 0.6, mix = 0.5, bypass = true },
    { name = "Compressor", type = EffectType.Compressor, threshold = -18.0, ratio = 4.0, attack = 0.01, release = 0.3, makeup = 0.0, knee = 0.0, bypass = true },
    { name = "Limiter",    type = EffectType.Limiter, threshold = -1.0, attack = 0.001, release = 0.1, bypass = true },
    { name = "NoiseGate",  type = EffectType.NoiseGate, threshold = -40.0, attack = 0.01, release = 0.1, bypass = true }
  },
  master = { gain = 1.0, tempo = 90 }
}
```

---

## Oscillator Parameters

| Parameter    | Type   | Description                            |
| ------------ | ------ | -------------------------------------- |
| `name`       | string | Display name of the oscillator         |
| `type`       | enum   | See `OscillatorType` below             |
| `transpose`  | number | Semitone offset                        |
| `detune`     | number | Fine-tune in cents                     |
| `mix`        | float  | Relative level (0.0–1.0)               |
| `phase`      | float  | Start phase (0.0–1.0)                  |
| `pwm`        | float  | Pulse width for PWM type               |
| `sync`       | bool   | Sync mode enabled                      |
| `fm_amount`  | float  | Amount of frequency modulation input   |


### OscillatorType (enum)

* `Sine`
* `Square`
* `Saw`
* `Triangle`
* `Noise`
* `SquarePWM`
* `HyperSaw`
* `ChaosNoise`
* `FM`  *(special modulator-only oscillator, no output)*

> `OscillatorType::FM` is used solely to supply FM input to other oscillators and is excluded from the final audio mix.


---

## LFO Parameters

| Parameter    | Type   | Description                  |
| ------------ | ------ | ---------------------------- |
| `name`       | string | Name of the LFO              |
| `waveform`   | enum   | See `LFOWaveform` below      |
| `rate`       | float  | Oscillation speed in Hz      |
| `depth`      | float  | Modulation depth             |
| `assignment` | enum   | See `LFOAssignment` below    |
| `bypass`     | bool   | If true, the LFO is bypassed |

### LFOWaveform (enum)

* `Sine`
* `Triangle`
* `Square`
* `Saw`

### LFOAssignment (enum)

* `None`
* `OscGain`
* `OscPitch`
* `FilterCutoff`
* `EffectMix`

---

## Filter Parameters

Each filter has:

* `name` (string)
* `type` (see `FilterType`)
* `bypass` (bool)

### FilterType (enum)

* `LowPass`
* `HighPass`
* `BandPass`
* `MoogLowPass`
* `Notch`

---

## EffectType (enum) & Parameters

All effect entries share:

* `name`, `type`, `bypass`, and usually a `mix` parameter.
  Additional parameters depend on type:

| EffectType   | Specific Fields                                                        |
| ------------ | ---------------------------------------------------------------------- |
| `Distortion` | `mode` (see `DistortionType`), `gain`, `bias`, `bitDepth`, `reduction` |
| `Chorus`     | `rate`, `depth`                                                        |
| `Delay`      | `time`, `feedback`                                                     |
| `Reverb`     | `size`, `damping`                                                      |
| `Flanger`    | `rate`, `depth`, `delay`                                               |
| `Rotary`     | `rate`, `depth`                                                        |
| `Compressor` | `threshold`, `ratio`, `attack`, `release`, `makeup`, `knee`            |
| `Limiter`    | `threshold`, `attack`, `release`                                       |
| `NoiseGate`  | `threshold`, `attack`, `release`                                       |

### DistortionType (enum)

* `HardClip`
* `Overdrive`
* `Bitcrush`
* `Fuzz`


---

## Envelope Parameters

| Parameter | Type  | Description             |
| --------- | ----- | ----------------------- |
| `attack`  | float | Attack time in seconds  |
| `decay`   | float | Decay time in seconds   |
| `sustain` | float | Sustain level (0.0–1.0) |
| `release` | float | Release time in seconds |

---

## Master Parameters

| Parameter | Type  | Description                     |
| --------- | ----- | ------------------------------- |
| `gain`    | float | Overall output level (0.0–1.0)  |
| `tempo`   | float | Global tempo in BPM             |
| `bpm`     | float | Alias for `tempo`, if preferred |

---

This preset system enables deep control over synthesis while remaining flexible and JSON-compatible. Scripting and modulation matrix integration are covered in later chapters.
