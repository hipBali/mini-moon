# Step-by-Step Example: Accelerating Ping-Pong Pan

This example demonstrates how to use an LFO to create a ping-pong style auto-pan effect with dynamic rate modulation. The result is a stereo motion that speeds up and slows down in cycles.

---

## Goal

Create an LFO that:

* Modulates the stereo `pan` of an oscillator or effect
* Speeds up gradually over time
* Then slows down again — creating a wave-like tempo breathing effect

---

## Setup

We’ll use:

* One main LFO for panning (`PingPongLFO`)
* One helper LFO (`SpeedModLFO`) that modulates the frequency of the first

---

## Full Lua Preset Snippet

```lua
lfo = {
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
        -- Remap from [-1..1] to [0.25..4.0] Hz
        local newFreq = 0.25 + (value + 1) * 1.875
        lfo.set{ name = "PingPongLFO", frequency = newFreq }
      end
    }
  }
}
```

---

## Explanation

* The **PingPongLFO** controls stereo position via `pan` using a sine waveform.
* The **SpeedModLFO** modulates its `frequency` between 0.25 Hz and 4 Hz in a triangle pattern.
* This results in an effect where the panning speeds up and slows down rhythmically.

---

## Tips for Customization

* Try using `LFOWaveform.Saw` on the SpeedModLFO for an accelerating drop effect.
* Replace `oscillator.set` with `effect.set` if you want to pan a chorus or delay module.
* You can dynamically change `depth` as well for motion shaping:

```lua
lfo.set{ name = "PingPongLFO", depth = 0.5 + 0.5 * value }
```

---

This example showcases how LFOs can be layered and interconnected to create evolving motion in a patch — and how `lfo.set()` allows real-time modulation of modulators themselves.
