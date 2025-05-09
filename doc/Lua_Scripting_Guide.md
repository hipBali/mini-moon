# Lua Scripting Guide

This guide covers scripting support for the synth engine, including real-time control, voice handling, step sequencing, modular patching, and live MIDI input.

---

## General Concepts

* Each preset script can define `init()`, `midi_cc = {}` handlers.
* Scripts are executed per preset load. Runtime behavior must be set up **in `init()` or at first MIDI input**.
* Global `sequenser(step)` is automatically called during internal step sequencing.

---

## Script Structure

### `init()`

Runs once when the preset is loaded.

```lua
init = function()
  master.set { tempo = 120, gain = 1.0 }
  ctl.setupSequencer {
    resolution = 16,
    startBar = 1,
    loopBars = 8,
    swing = 0.0,
    mute = false,
    metronome = false
  }
end
```

### `midi_cc = {}`

Table for MIDI event handling.

```lua
midi_cc = {
  note_on = function(note, vel, channel)
    ctl.noteOn(note, vel)
  end,

  note_off = function(note,channel)
    ctl.noteOff(note)
  end,

  control = function(channel, cc, value)
    print("Received CC", cc, value)
  end
}
```

---

## ctl API Overview

| Function                            | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `ctl.noteOn(note, velocity)`        | Trigger internal voice (0–127)                               |
| `ctl.noteOff(note)`                 | Release note                                                 |
| `ctl.getStep()`                     | Get current internal step counter                            |
| `ctl.setupSequencer{...}`          | Configure step resolution, loop, swing, mute, metronome      |
| `ctl.loadModule(name)`             | Load JSON preset into `modules` table                        |
| `ctl.isMuted()`                     | Query current mute state (if registered)                     |
| `ctl.sendMidi(table)` *(planned)*  | Send MIDI message out (note_on, control, etc.)               |

---

## Sequencing with `sequenser(step)`

Define a global function to receive step callbacks:

```lua
sequenser = function(step)
  print("Step", step)
end
```

This is automatically called based on the `setupSequencer` parameters. Example:

```lua
ctl.setupSequencer{
  resolution = 16,
  loopBars = 4,
  swing = 0.5,
  metronome = true,
  mute = false
}
```

Metronome clicks are played internally on bar boundaries and beats depending on configuration.

---

## Modules via JSON

```lua
ctl.loadModule("001-piano")
```

Loads `presets/001-piano.json` and applies the `modules` definition.

---

## Built-in Libraries (`scripts/`)

### `note_sync`

Synchronize note triggering with internal step timing.

```lua
local sync = require "scripts.note_sync"
sync.set_steps_per_measure(16)
sync.queue_note(note, vel, ctl.getStep() + 1)
```

### `loop_play`

Simple step sequencer based on Giorgio Moroder style patterns.

```lua
local loop = require "scripts.loop_play"
loop.set_pattern { 0, 7, 10, 7 }
loop.set_bar_length(2)
loop.set_steps_per_beat(4)
loop.set_octave_shift(1)
```

### `chord_lib`

Generates chords from scale, inversion, spread, etc.

```lua
local chord = chord_lib.generate {
  root = 48, scale = "minor", notes = 4,
  reverse = 0, inversion = 1, spread = 1, key = "Eb"
}
```

---

## Example: Chord + Loop Split

```lua
midi_cc = {
  note_on = function(note, vel)
    if note <= 59 then
      local chord = chord_lib.generate { root = note, scale = "minor", notes = 4 }
      for i, n in ipairs(chord) do if i > 1 then ctl.noteOn(n, vel) end end
    else
      loop.set_pattern { 0, 7, 10, 7 }
      loop.start(note, vel, ctl.getStep())
    end
  end,
  note_off = function(note)
    ctl.noteOff(note)
    loop.stop()
  end
}

init = function()
  master.set { tempo = 120 }
  ctl.setupSequencer {
    resolution = 16,
    loopBars = 4,
    metronome = true,
    mute = false
  }
end
```

---

## MIDI Controller Input (CC)

Handled in:

```lua
midi_cc = {
  control = function(cc, value)
    -- value: 0–127
    if cc == 1 then  -- Mod wheel
      print("Modulation: ", value)
    end
  end
}
```

Future plans:

* `ctl.mapCC(cc, function)` for mapping callbacks directly

---

## Tips

* Use `ctl.getStep()` for quantized sequencing
* Define `sequenser(step)` once globally in each script
* Use `ctl.setupSequencer{}` for full control over step behavior
* Modularize common logic into `scripts/*.lua`

---

## Coming Soon

* `ctl.sendMidi { type = "note_on", note = 60, velocity = 100, channel = 1 }`
* `ctl.listMidiPorts()` and `ctl.setMidiOutPort()`
* `ctl.mapCC()` and `ctl.setParam()` for real-time control

---
