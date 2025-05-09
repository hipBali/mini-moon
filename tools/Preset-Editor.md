# Mini-Moon Preset Editor

This editor supports creating, editing, loading, and saving Mini-Moon presets in **JSON format**. 

---

## Supported Modules (under `modules` key)

Each preset JSON contains the following module types:

- `oscillator[]` – synthesizer oscillators
- `lfo[]` – low frequency oscillators
- `envelope` – ADSR envelope
- `filter[]` – filters
- `effect[]` – audio effects (e.g., reverb, delay, distortion)
- `master` – global parameters (gain, tempo, pitch, glide, etc.)

---

## Editor Functionality

### Module Reordering
- `filter` and `effect` entries can be moved up (▲) or down (▼)

### Deletion
- All modules (except `envelope` and `master`) can be individually deleted (✖)

### Addition
- You can dynamically add new `oscillator`, `lfo`, `filter`, and `effect` modules

### Indexing
- Module names are automatically indexed (`OSC-1`, `FLT-2`, etc.)
- Counters resync after loading a preset to avoid duplicate names

### Enum Values
- Types are stored as enums in the form of `EnumPrefix.Value`, e.g.:
  - `OscillatorType.Sine`
  - `FilterType.LowPass`
  - `EffectType.Reverb`
  - `DistortionType.Fuzz`
  - `LFOAssignment.OscPitch`
  - `LFOWaveform.Triangle`

---

## Example JSON Structure

```json
{
  "modules": {
    "oscillator": [
      {
        "name": "OSC-1",
        "type": "OscillatorType.Square",
        "gain": 1,
        "detune": 0,
        "pan": 0
      }
    ],
    "lfo": [
      {
        "name": "LFO-1",
        "assignment": "LFOAssignment.OscPitch",
        "target": "OSC-1",
        "frequency": 2,
        "depth": 0.5,
        "waveform": "LFOWaveform.Sine"
      }
    ],
    "envelope": {
      "attack": 0.1,
      "decay": 0.3,
      "sustain": 0.7,
      "release": 0.9
    },
    "filter": [
      {
        "name": "FLT-1",
        "type": "FilterType.LowPass",
        "cutoff": 1000,
        "resonance": 0.5
      }
    ],
    "effect": [
      {
        "name": "FX-1",
        "type": "EffectType.Distortion",
        "mode": "DistortionType.Fuzz",
        "mix": 0.8,
        "rate": 2.0
      }
    ],
    "master": {
      "gain": 1.0,
      "pan": 0,
      "pitch": 0,
      "tempo": 120,
      "tremolo": 0.3,
      "glide": {
        "time": 0.05,
        "enabled": true
      }
    }
  }
}
```

---

## Workflow Summary

- On load: UI is updated and module counters are synchronized
- On save: all data is exported under a single `modules` JSON key
- Preview panel: updated in real time after any change

---

_Last updated for the JSON-only version of Mini-Moon Preset Editor._
