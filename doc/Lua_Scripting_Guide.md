# Lua Scripting Guide

This guide describes scripting capabilities for the synth engine, including preset control, modular patching, sequencing, and MIDI handling.

---

## General Concepts

* Each script is loaded on preset switch.
* Use `init()` to configure modules, sequencing, or MIDI behavior.
* Global functions such as `sequenser(step)` or tables like `midi.map` provide integration points.
* All timing and control resolution is driven by the internal step engine.

---

## Script Structure

### `init()`

Runs once when the preset is loaded.

```lua
init = function()
  master.set { tempo = 120, gain = 0.8 }
  ctl.setupSequencer {
    resolution = 16,
    loopBars = 4,
    swing = 0.2,
    mute = false,
    metronome = true
  }
end
```

---

## Step Sequencing

Define a global function to respond to step events.

```lua
sequenser = function(step)
  print("Step:", step)
end
```

Step timing is controlled by `ctl.setupSequencer`.

---

## MIDI Input Handling

### MIDI Maps

MIDI CC and pitchbend input are mapped using `midi.map`.

```lua
midi = {
  map = {
    { cc = 20, val = 127, handler = ctl.savePreset },
    { cc = 19, val = 127, handler = ctl.loadPreset },
    { cc = 17, val = 127, handler = ctl.nextPreset },
    { cc = 16, val = 127, handler = ctl.prevPreset },
    { cc = 7, channel = 1, handler = function(val) ctl.pots[1] = val end }
  },

  noteOn = function(note, vel, chn)
    ctl.noteOn(note, vel)
  end,

  noteOff = function(note, chn)
    ctl.noteOff(note)
  end
}
```

The `map` table entries accept:

| Field     | Type     | Description                   |
| --------- | -------- | ----------------------------- |
| `cc`      | number   | MIDI CC number (0–127)        |
| `val`     | number   | Optional value match          |
| `channel` | number   | Optional channel match (0–15) |
| `handler` | function | Function to call on match     |

If `handler` is missing or `nil`, the entry is skipped.

---

## ctl API

| Function                  | Description                                  |
| ------------------------- | -------------------------------------------- |
| `ctl.noteOn(note, vel)`   | Trigger voice                                |
| `ctl.noteOff(note)`       | Release voice                                |
| `ctl.getStep()`           | Current step                                 |
| `ctl.setupSequencer{...}` | Configure resolution, swing, metronome, mute |
| `ctl.loadModule(name)`    | Load a JSON patch from `presets/`            |
| `ctl.nextPreset()`        | Load next preset                             |
| `ctl.prevPreset()`        | Load previous preset                         |
| `ctl.savePreset()`        | Save live preset                             |
| `ctl.loadPreset()`        | Load last saved live preset                  |

---

## JSON Modules

```lua
ctl.loadModule("010-deep_bass_poly")
```

This loads `presets/010-deep_bass_poly.json`.

---

## Sequencer Example

```lua
init = function()
  ctl.setupSequencer {
    resolution = 16,
    loopBars = 8,
    swing = 0.4,
    mute = false,
    metronome = true
  }
end

sequenser = function(step)
  print("At step", step)
end
```

---

## Tips

* Define `midi.noteOn` and `midi.noteOff` for custom routing.
* Use `midi.map` for dynamic controller mapping.
* Configure everything in `init()`, only one sequencer is supported at once.
* Use `ctl.getStep()` for step-aware control or quantization.

---
