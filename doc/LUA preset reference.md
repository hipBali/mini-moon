# Mini-Moon Lua Preset Reference

This document describes how to define full presets for Mini-Moon using Lua scripting. It extends JSON-style configuration with runtime control and callback logic.

---

## Preset Entry Point

Every Lua preset must define:

```lua
modules = { ... }
init = function() ... end
```

* `modules` — synth module configuration (same as JSON)
* `init()` — optional function run once on load (e.g. to install callbacks)

---

## Example Structure

```lua
modules = {
  oscillator = {
    { name = "OSC-1", type = OscillatorType.Saw, gain = 1.0, pan = 0.0 }
  },
  filter = {
    { name = "LPF", type = FilterType.LowPass, cutoff = 800.0 }
  },
  envelope = {
    attack = 0.01, decay = 0.2, sustain = 0.8, release = 0.3
  },
  lfo = {
    {
      name = "LFO-1",
      waveform = LFOWaveform.Sine,
      frequency = 2.0,
      depth = 1.0,
      assignment = LFOAssignment.OscGain,
      target = "OSC-1",
      callback = {
        interval = 8,
        func = function(name, value, target)
          oscillator.set{ name = target, gain = 0.5 + 0.5 * value }
        end
      }
    }
  },
  dynamicEnvelope = {
    {
      points = {
        { t = 0.0, v = 0.0 },
        { t = 0.1, v = 1.0 },
        { t = 1.0, v = 0.0 }
      },
      callback = {
        interval = 4,
        loop = true,
        func = function(v)
          master.set{ gain = v }
        end
      }
    }
  },
  master = {
    gain = 1.0,
    tempo = 120,
    portamento = 0.0
  }
}

init = function()
  print("[Lua] Preset initialized.")
  ctl.noteOn(60, 100)  -- (Optional test trigger)
end
```

---

## Best Practices

* Avoid redundant assignment + callback in same LFO.
* Use `init()` for dynamic logic (e.g. callback registration).
* Use `ctl.getNoteTime(n)` to drive `dynamicEnvelope` timing.
* Use `oscillator.set{}` or `master.set{}` inside callbacks.
* Prefer readable `enum` names (e.g. `OscillatorType.Sine`).

---

## Available Module Keys

* `oscillator` → array of synth voices
* `filter` → serial filter chain
* `effect` → serial effect chain
* `envelope` → global ADSR table
* `lfo` → array with optional `callback`
* `dynamicEnvelope` → array of timed control shapes
* `master` → global parameters

---

## Mixed Preset Mode

Mini-Moon also supports loading a base preset (JSON or Lua) and modifying it on the fly.

### Example

```lua
ctl.loadModule("warm_lead")

modules.master = {
  gain = 0.8,
  tempo = 96,
  portamento = 0.03
}

init = function()
  print("Hybrid preset loaded and modified.")
end
```

### Notes

* `ctl.loadModule(name)` loads an existing preset from disk.
* You can overwrite any `modules` fields afterward.
* `init()` is still used for scripting or registering callbacks.

---

## End of Reference
