# Modulation Matrix and LFO Callback System

Mini-Moon enables advanced sound shaping via a dynamic, scriptable modulation matrix. This system combines classical LFO-based modulation assignments with arbitrary Lua callbacks, offering powerful and expressive control.

---

## 🎛 Standard LFO Assignments

Each LFO can be assigned to a built-in modulation path using the `assignment` field:

```lua
assignment = LFOAssignment.OscPitch
```

### 🎚 Supported `LFOAssignment` Enums:

* `None` (no built-in modulation)
* `OscPitch`
* `OscGain`
* `FilterCutoff`
* `EffectMix`
* `Pan`

> When using `None`, the LFO will not affect any signal path by default — perfect for custom scripting via callbacks.

---

## ⏱ Sync to Tempo

Each LFO can optionally be synchronized to the host tempo by setting:

```lua
syncToTempo = true,
division = 0.5  -- half = eighth note, 1.0 = quarter, 2.0 = half, etc.
```

When `syncToTempo` is true, the `frequency` is ignored and `division` is used instead for rhythmic modulation.

---

## 🧠 Scripted Modulation via LFO Callbacks

To build a modulation matrix in Lua, define a `callback` table inside each LFO. This allows full control over any parameter:

```lua
callback = {
  interval = 16,  -- execution frequency (steps)
  func = function(lfoName, value, target)
    effect.set{ name = target, mix = 0.5 + 0.5 * value }
  end
}
```

### 🔢 Callback Arguments

* `lfoName` – the LFO's `name`
* `value` – current LFO output in range `[-1.0 .. 1.0]`
* `target` – the `target` field defined in the LFO block

This mechanism forms the basis of a programmable modulation matrix.

> ⚠️ Avoid redundancy: do not combine a standard assignment with a callback affecting the same parameter.

---

## 🧩 Target-Aware Routing

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

## 🎨 Creative LFO Callback Ideas

### 🎚️ 1. Animated Effect Parameters

```lua
effect.set{ name = "Reverb", mix = 0.4 + 0.3 * value }
effect.set{ name = "Flanger", delay = 0.5 + 1.0 * value }
effect.set{ name = "Chorus", rate = 0.1 + 0.05 * value, depth = 0.2 + 0.3 * value }
```

### 🔊 2. Panning, Vibrato, Tremolo

```lua
oscillator.set{ name = "OSC-1", pan = value }
master.set{ gain = 0.5 + 0.5 * value }
oscillator.set{ name = "OSC-1", pwm = 0.4 + 0.3 * value }
```

### 🧪 3. Combined Modulations

```lua
oscillator.set{ name = "OSC-2", fm_amount = 0.2 + 0.2 * value }
effect.set{ name = "Distortion", bitDepth = math.floor(4 + 4 * (1 - value)) }
```

### 🌪 4. Global AutoPan with Dynamic Speed

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

Creates a breathing ping-pong effect where the stereo pan accelerates and decelerates.

### 🎯 5. Dynamic Target Routing

```lua
callback = {
  interval = 8,
  func = function(name, v)
    effect.set{ name = name, mix = 0.3 + 0.3 * v }
  end
}
```

---

## 🧠 Modulation Shaping Techniques

You can use any Lua math and logic in a callback:

```lua
local shaped = math.sin(value * math.pi)
if value > 0 then
  filter.set{ name = target, cutoff = 2000 + shaped * 1000 }
end
```

Great for adding soft curves, symmetry, or gated behaviors.

---

## 🛠 Live Parameters You Can Modulate

### 🎚 Oscillator

* `mix`, `pwm`, `fm_amount`, `phase`, `pan`, `transpose`

### 🔊 Filter

* `cutoff`, `resonance`, `gain`, `slope`, `bypass`

### 🌊 Effect

* `mix`, `depth`, `rate`, `delay`, `feedback`, `threshold`, `bitDepth`, `pan`, etc.

### 🎧 Master

* `gain`, `tempo`, `pan`

> All modules support both `module.set{}` (live only) and `module.update{}` (live + preset).

Example:

```lua
lfo.set{ name = "LFO-1", depth = 0.5 }
filter.update{ name = "LPF", cutoff = 1200 }
```

---

## 🚫 Known Modulation Limitations

While most parameters are modulateable in real time, a few are statically applied at note-on time and do not respond to live updates. Notable examples:

* `oscillator.pan` → only applied during `noteOn()` (per-voice init)
* `oscillator.gain` → voice gain is set at note start
* Envelope parameters (`attack`, `release`, etc.) cannot be updated mid-note
* Some filter/effect parameters may not support smooth automation depending on implementation

For consistent real-time control, prefer modulating parameters on:

* `master` (global)
* `effect` (independent stereo/FX path)
* `filter` (in many cases realtime-safe)

---

## ⚠ Known Limitations

* Overlapping modulations to the same parameter from multiple LFOs may cause unpredictable results.
* If a callback targets an invalid module name or parameter, the call may silently fail or be ignored.
* Callback execution should be efficient — heavy logic in high-rate intervals (e.g. `interval = 1`) can affect timing.
* Using both `assignment` and `callback` on the same parameter is **not recommended**.

---

## 🌐 Example: Dual LFO Setup

```lua
lfo = {
  {
    name = "VibratoLFO",
    assignment = LFOAssignment.OscPitch,
    frequency = 6.0,
    depth = 0.02,
    waveform = LFOWaveform.Sine
  },
  {
    name = "ManualPanLFO",
    assignment = LFOAssignment.None,
    target = "FX-1",
    frequency = 0.5,
    depth = 1.0,
    waveform = LFOWaveform.Triangle,
    callback = {
      interval = 32,
      func = function(lfoName,v,target)
        effect.set{ name = target, pan = v }
      end
    }
  }
}
```

---

## 🧷 Summary

| Feature            | Description                                    |
| ------------------ | ---------------------------------------------- |
| LFO assignment     | Built-in modulation path                       |
| `callback.func()`  | Arbitrary modulation logic                     |
| `target`           | Named module used in the callback              |
| `interval`         | How often the callback executes (in steps)     |
| Value range        | `[-1.0 .. 1.0]` LFO output                     |
| Modulation shaping | Math, gating, logic supported in Lua           |
| Dynamic access     | `.set` and `.update` supported for all modules |
| Limitations        | Avoid redundant modulations or invalid targets |

This system turns every LFO into a programmable modulation source — a true flexible matrix, defined in pure Lua.
