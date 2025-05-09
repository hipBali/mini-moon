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
    "oscillator": [],
    "lfo": [],
    "envelope": {
      "attack": 0.1,
      "decay": 0.3,
      "sustain": 0.7,
      "release": 0.9
    },
    "filter": [],
    "effect": [],
    "master": {
      "gain": 1,
      "pan": 0,
      "pitch": 0,
      "tempo": 120,
      "tremolo": 0,
      "glide": {
        "time": 0,
        "enabled": false
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

