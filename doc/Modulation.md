# Modulation Matrix and LFO Callback System

Mini-Moon enables advanced sound shaping via a dynamic, scriptable modulation matrix. This system combines classical LFO-based modulation assignments with arbitrary Lua callbacks, offering powerful and expressive control.

---

## Standard LFO Assignments

Each LFO can be assigned to a built-in modulation path using the `assignment` field:

```lua
assignment = LFOAssignment.OscPitch
```

### Supported `LFOAssignment` Enums:

* `None` (no built-in modulation)
* `OscPitch`
* `OscGain`
* `OscPWM`
* `FilterCutoff`
* `FilterResonance`
* `Pan`
* `Gain`
* `Pitch`
* `Tremolo`
* `EnvAttack`    

> When using `None`, the LFO will not affect any signal path by default — perfect for custom scripting via callbacks.

---

## Sync to Tempo

Each LFO can optionally be synchronized to the host tempo by setting:

```lua
syncToTempo = true,
division = 0.5  -- half = eighth note, 1.0 = quarter, 2.0 = half, etc.
```

When `syncToTempo` is true, the `frequency` is ignored and `division` is used instead for rhythmic modulation.

---

## Scripted Modulation via LFO Callbacks

To build a modulation matrix in Lua, define a `callback` table inside each LFO. This allows full control over any parameter:

```lua
callback = {
  interval = 16,  -- execution frequency (frames)
  func = function(lfoName, value, target)
    effect.set{ name = target, mix = 0.5 + 0.5 * value }
  end
}
```

### Callback Arguments

* `lfoName` – the LFO's `name`
* `value` – current LFO output in range `[-1.0 .. 1.0]`
* `target` – the `target` field defined in the LFO block

> The `interval` defines how often (in frames) the callback is invoked. For example, `interval = 4` means the callback runs every 4 audio frames. Use higher values (e.g., 16–64) for smooth modulation.

---

## Dynamic Envelope Modulation

In addition to LFOs, Mini-Moon supports a single scriptable envelope generator: `dynamicEnvelope`.

This envelope triggers automatically when the first `noteOn()` occurs, and runs while any notes are active.

```lua
dynamicEnvelope.set{
  points = {
    { t = 0.0, v = 0.0 },
    { t = 0.1, v = 1.0 },
    { t = 0.5, v = 0.8, mode = "log" },
    { t = 1.0, v = 0.0, mode = "sine" }
  },
  callback = {
    interval = 8,
    loop = false,
    func = function(value)
      oscillator.set{ name = "OSC-1", pitch = -12 + value * 24 }
    end
  }
}
```

### Point Modes

Each envelope point can specify a `mode`:

* `"linear"` (default)
* `"exp"` – exponential ramp
* `"log"` – logarithmic decay
* `"sine"` – smooth curve

Use `loop = true` to restart after the final point:

```lua
dynamicEnvelope.set{ loop = true, ... }
```

> The envelope runs from the moment of first note-on and stops when the last note-off occurs.

---

## Target-Aware Routing

Each LFO can include a `target` field, used freely inside its callback:

```lua
{
  name = "ManualPanLFO",
  assignment = LFOAssignment.None,
  target = "FX-1",
  frequency = 0.5,
  depth = 1.0,
  waveform = LFOWaveform.Triangle,
  callback = {
    interval = 32,
    func = function(lfoName, v, target)
      effect.set{ name = target, pan = v }
    end
  }
}
```

---

## PWM Modulation (Pulse Width Modulation)

PWM dynamically alters the duty cycle of square waveforms to produce evolving textures.

### Supported Oscillator Types

* `OscillatorType::SquarePWM`

### Modulation Targets

* `pulseWidth` — direct control of duty cycle (0.0–1.0, clamped internally to 0.01–0.99)
* `pulseWidthMod` — per-oscillator modulation sensitivity (used with `LFOAssignment::OscPWM`)

### LFO Routing via Assignment

Use `assignment = LFOAssignment.OscPWM` and (optionally) a `target` field to apply PWM to specific oscillators:

```lua
{
  name = "PWM-LFO",
  waveform = LFOWaveform.Sine,
  frequency = 1.0,
  depth = 1.0,
  assignment = LFOAssignment.OscPWM,
  target = "PWM-1"  -- optional
}
```

If `target` is omitted, all `SquarePWM` oscillators receive the modulation.

---

## Creative Modulation Ideas

### 1. Animated Effect Parameters

```lua
effect.set{ name = "Reverb", mix = 0.4 + 0.3 * value }
effect.set{ name = "Flanger", delay = 0.5 + 1.0 * value }
effect.set{ name = "Chorus", rate = 0.1 + 0.05 * value, depth = 0.2 + 0.3 * value }

```

---

### 2. Panning, Vibrato, Tremolo

```lua
oscillator.set{ name = "OSC-1", pan = value }
master.set{ gain = 0.5 + 0.5 * value }
oscillator.set{ name = "OSC-1", pwm = 0.4 + 0.3 * value }
```

### 3. Combined Modulations

```lua
oscillator.set{ name = "OSC-2", fmAmount = 0.2 + 0.2 * value }
effect.set{ name = "Distortion", bitDepth = math.floor(4 + 4 * (1 - value)) }
```

### 4. Global AutoPan with Dynamic Speed

```lua
{
  name = "PingPongLFO",
  assignment = LFOAssignment.None,
  target = "OSC-1",
  frequency = 1.0,
  depth = 1.0,
  waveform = LFOWaveform.Sine,
  callback = {
    interval = 8,
    func = function(name, value, target)
      master.set{ pan = value }
    end
  }
},
{
  name = "SpeedModLFO",
  assignment = LFOAssignment.None,
  frequency = 0.05,
  depth = 1.0,
  waveform = LFOWaveform.Triangle,
  callback = {
    interval = 16,
    func = function(name, value)
      local newFreq = 0.25 + (value + 1) * 1.875
      lfo.set{ name = "PingPongLFO", frequency = newFreq }
    end
  }
}
```

### 5. Dynamic Target Routing

```lua
callback = {
  interval = 8,
  func = function(name, v)
    effect.set{ name = name, mix = 0.3 + 0.3 * v }
  end
}
```

### 6. Envelope-Controlled FM or Sync Sweep

```lua
callback = {
  interval = 8,
  func = function(value)
    oscillator.set{ name = "OSC-2", fmAmount = value * 0.8 }
    oscillator.set{ name = "OSC-1", phase = value }
  end
}
```

---

## Modulation Shaping Techniques

You can use any Lua math and logic in a callback:

```lua
local shaped = math.sin(value * math.pi)
if value > 0 then
  filter.set{ name = target, cutoff = 2000 + shaped * 1000 }
end
```

---

## Live Parameters You Can Modulate

### Oscillator

* `mix`, `pwm`, `fmAmount`, `phase`, `pan`, `transpose`

### Filter

* `cutoff`, `resonance`, `gain`, `slope`, `bypass`

### Effect

* `mix`, `depth`, `rate`, `delay`, `feedback`, `threshold`, `bitDepth`, `pan`, etc.

### Master

* `gain`, `tempo`, `pan`

> All modules support both `module.set{}` (live only) and `module.update{}` (live + preset).

```lua
lfo.set{ name = "LFO-1", depth = 0.5 }
filter.update{ name = "LPF", cutoff = 1200 }
```

---

## Known Modulation Limitations

* Overlapping modulations to the same parameter from multiple LFOs may cause unpredictable results.
* If a callback targets an invalid module name or parameter, the call may silently fail or be ignored.
* Callback execution should be efficient — heavy logic in high-rate intervals (e.g. `interval = 1`) can affect timing.
* Using both `assignment` and `callback` on the same parameter is **not recommended**.
* `oscillator.pan` and `oscillator.gain` are only applied per voice at `noteOn()`.
* Envelope parameters cannot be changed mid-note.

---

## Summary

| Feature            | Description                                   |
| ------------------ | --------------------------------------------- |
| LFO assignment     | Built-in modulation path                      |
| `callback.func()`  | Arbitrary modulation logic                    |
| `target`           | Named module used in the callback             |
| `interval`         | How often the callback executes (in frames)   |
| `value` range      | `[-1.0 .. 1.0]` LFO output                    |
| DynamicEnvelope    | Time-based one-shot or looping envelope       |
| `points[]`         | Envelope curve points with optional `mode`    |
| Modulation shaping | Math, logic, interpolation supported in Lua   |
| Dynamic access     | `.set` (live), `.update` (live + preset)      |
| Limitations        | Avoid redundancy and unsupported live targets |

---

This system turns every LFO and envelope into a programmable modulation source — a true flexible matrix, defined in pure Lua.
