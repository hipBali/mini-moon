# Lua Scripting Guide for PolySynth

This document describes the scripting interface available for PolySynth via Lua. It includes configurable modules, MIDI handling, the preset structure, and examples.

---

## 📦 Preset Structure

Every preset must define a global `modules` table. This contains one or more of the following submodules:

```lua
modules = {
  oscillator = { ... },
  envelope = { ... },
  filter = { ... },
  lfo = { ... },
  distortion = { ... },
  delay = { ... },
  reverb = { ... },
  flanger = { ... },
  rotary = { ... },
  chorus = { ... },
  compressor = { ... },
  limiter = { ... },
  noisegate = { ... },
  glide = { ... },
  master = { ... }
}
```

### Optional: `init()` function

An optional `init()` function may be defined. It is executed after the preset is loaded, and can be used to register callbacks or change values dynamically.

```lua
function init()
  master.set{ tempo = 120 }
  modulation.set{ callback = myCallback }
end
```

---

## 🔊 Modules and Parameters

### Oscillator

```lua
oscillator = {
  { name = "OSC-1", type = OscillatorType.Sine, gain = 0.5, detune = 0.0, pan = 0.0, transpose = 0 },
  ...
}
```

- `name`: string (must be unique)
- `type`: `OscillatorType` (e.g., `Sine`, `SquarePWM`, `Triangle`, `Sawtooth`)
- `gain`: float
- `detune`: float
- `pan`: float (-1 to 1)
- `transpose`: semitone offset

### Envelope

```lua
envelope = { attack = 0.01, decay = 0.1, sustain = 0.8, release = 0.5 }
```

### Filter

```lua
filter = {
  { name = "LPF", type = FilterType.LowPass, cutoff = 1200.0, resonance = 0.7, bypass = false },
  ...
}
```

Supported filter types:
- `LowPass`, `HighPass`, `BandPass`, `MoogLowPass`, `Notch`

### LFO

```lua
lfo = {
  {
    name = "Vibrato",
    waveform = LFOWaveform.Triangle,
    assignment = LFOAssignment.OscPitch,
    frequency = 5.0,
    depth = 0.2,
    bypass = false
  }
}
```

### FX Modules

Each FX module supports `bypass = true|false` and has individual parameters. Examples:

```lua
delay = { time = 0.3, feedback = 0.6, mix = 0.5, bypass = false }
reverb = { size = 0.6, damping = 0.5, mix = 0.3, bypass = false }
rotary = { rate = 1.2, depth = 0.8, mix = 0.4, bypass = false }
```

### Glide

```lua
glide = { time = 0.2, bypass = false }
```

### Master

```lua
master = { gain = 0.85, pan = 0.5 }
```

---

## 🎛 MIDI Handling

### 1. `midi_cc` and `global_midi_cc`

These tables assign controller numbers to functions.

```lua
global_midi_cc = {
  [16] = function() ctl:prevPreset() end,
  [17] = function() ctl:nextPreset() end
}

midi_cc = {
  [30] = function(value)
    local pan = -1.0 + value / 64.0
    master.set{ pan = pan }
  end
}
```

### 2. `midi_cc_mode`

Defines controller behavior: `"absolute"` or `"button"`.

```lua
midi_cc_mode = {
  [16] = "button",
  [30] = "absolute"
}
```

### 3. `midi_events`

```lua
midi_events = {
  mod_wheel = function(value) print("Mod:", value) end,
  pitch_bend = function(value) oscillator.set{ "OSC-1", pan = value } end
}
```

---

## 💾 Preset Saving

A modified preset state can be saved with timestamps using:

```lua
ctl:savePreset()
```

The file will be saved under `live-presets/<preset>_YYYYMMDD_HHMMSS.lua`.

---

## 🧪 Example Preset

```lua
modules = {
  oscillator = {
    { name = "SINE", type = OscillatorType.Sine, gain = 0.6, pan = 0.0 }
  },
  envelope = {
    attack = 0.02, decay = 0.4, sustain = 0.6, release = 1.0
  },
  filter = {
    { name = "LPF", type = FilterType.LowPass, cutoff = 900, resonance = 0.7 }
  },
  reverb = {
    size = 0.4, damping = 0.3, mix = 0.2
  },
  master = {
    gain = 0.9, pan = 0.5
  }
}

function init()
  master.set{ tempo = 90 }
end
```

---

## ℹ️ Notes

- All oscillator and LFO names must be unique.
- If `modules` is missing, the preset is rejected.
- Enum values (e.g. `OscillatorType.Sine`) must be available and are automatically registered.