# Mini-Moon Configuration Reference

This document describes the global configuration required by the Mini-Moon synth engine. These values are typically defined in `config.lua`, and are required for the engine to boot and operate correctly.

> `config.lua` is mandatory — the synth will not run without it.

---

## global\_config

```lua
global_config = {
  sampleRate = 44100  -- or 48000, 96000 depending on system
}
```

Sets the global sample rate for the audio engine. This must match the backend capabilities and typically aligns with the system default.

---

## presets

```lua
presets = {
  "001-sample",
  "002-bitcrush_bass",
  "003-dual_lfo_fx",
  "004-filter_sweep_drone",
  "005-glide_bass",
  "006-pwm_strings",
  "007-reso_pluck",
  "008-sync_lead",
  "009-tremolo_keys",
  "010-tremolo_master",
  "011-warm_pad"
}
```

This is the preset playlist — it defines the available patches for the engine.

Used by:

* `ctl:nextPreset()`
* `ctl:prevPreset()`
* `ctl:savePreset()` stores to the currently active entry (if allowed)

> At least one preset must be listed, or the engine will have nothing to load.

---

## velocity\_config

```lua
velocity_config = {
  thresholds = { 0.0, 0.2, 0.4, 0.6, 0.8, 1.01 },
  outputs    = { 0.2, 0.4, 0.6, 0.75, 0.9, 1.0 }
}
```

Defines velocity mapping. The `thresholds` list specifies input boundaries (normalized 0..1), while `outputs` provides mapped values. These arrays must:

* Be the same length
* Be monotonically increasing
* Cover 0.0 → 1.0

Used by `synth:mapVelocity()` during `noteOn()`.

---

## global\_midi\_cc

```lua
global_midi_cc = {
  control = function(channel, cc, value)
    if cc == 18 then ctl:savePreset()
    elseif cc == 17 then ctl:nextPreset()
    elseif cc == 16 then ctl:prevPreset()
    else return true end
    return false
  end
}
```

This callback is checked before any preset-level `midi_cc.control`. If it returns `false`, the event is consumed.

Use this for:

* Global CC remapping
* Preset control via MIDI knobs
* System-wide functions (save, next, prev, etc.)

---

## Summary

| Section           | Required | Purpose                                  |
| ----------------- | -------- | ---------------------------------------- |
| `global_config`   | yes      | Sets audio system parameters             |
| `presets`         | yes      | Defines available patches                |
| `velocity_config` | optional | Controls MIDI velocity mapping           |
| `global_midi_cc`  | optional | Provides global MIDI controller handling |

All configuration values must be valid Lua and defined at global scope in `config.lua`.

If this file is missing or malformed, the engine will terminate with an error.
