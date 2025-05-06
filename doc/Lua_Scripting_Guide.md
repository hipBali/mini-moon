# Mini-Moon Scripting API Reference (Lua)

This document describes how to define interactive, MIDI-reactive, and time-based behaviors in Mini-Moon presets using embedded Lua scripting.

---

## Where to Define Scripts

Scripts are defined directly within Lua-based presets or `.lua` files attached to hybrid presets. Scripting logic is handled through two main interfaces:

* The `midi_cc` table for MIDI-triggered logic
* The global `ctl` object for control and triggering API

---

## MIDI Callback Table: `midi_cc`

Define this table at the top level of your preset:

```lua
midi_cc = {
  note_on = function(note, velocity)
    -- Called when a MIDI note-on is received
  end,

  note_off = function(note)
    -- Called when a MIDI note-off is received
  end,

  control = function(cc, value)
    -- Called on generic MIDI CC input
    return true  -- allow fallback
  end,

  pitch_bend = function(value)
    -- Called on pitch bend event (0.0 - 1.0 normalized)
  end,

  mod_wheel = function(value)
    -- Called when CC1 (mod wheel) changes
  end
}
```

Each callback is optional. If not defined, it is skipped silently.

---

## Control API: `ctl` Object

The global `ctl` object exposes callable functions and event registration for scripting:

### Note trigger functions

```lua
ctl.noteOn(note, velocity)
ctl.noteOff(note)
```

These trigger internal note playback and can be used in arpeggiators, sequencers, etc.

### Step sequencer callback registration

```lua
ctl.onStep(function(step_index)
  -- Called once per step advance
end)
```

Only one function can be active at a time. Calling it again replaces the previous.

---

## Per-LFO Callback Support

```lua
modules = {
  lfo = {
    {
      name = "LFO-1",
      waveform = LFOWaveform.Sine,
      callback = function(value)
        -- value ∈ [-1.0 .. 1.0]
      end
    }
  }
}
```

LFO callbacks enable per-cycle logic like automation or shaping.

---

## Full Arpeggiator Example

```lua
local arp = {
    active = false,
    base = 48,  -- C3
    velocity = 100,
    index = 1,
    notes = { 0, 3, 7, 10 },  -- Cm7
    lastStep = -1,
    lastNote = nil,
    playedNotes = {}
}

midi_cc = {
  note_on = function(note, vel)
    arp.active = true
    arp.base = note
    arp.velocity = vel
    arp.index = 2
    arp.playedNotes[#arp.playedNotes + 1] = note
  end,

  note_off = function(note)
    if arp.active and note == arp.base then
      arp.active = false
      for _, n in ipairs(arp.playedNotes) do
        ctl.noteOff(n)
      end
      arp.playedNotes = {}
    end
  end,
}

ctl.onStep(function(step)
  if not arp.active or step == arp.lastStep then return end
  arp.lastStep = step

  local newNote = arp.base + arp.notes[arp.index]
  local lastNote = arp.playedNotes[#arp.playedNotes]
  if lastNote and lastNote ~= newNote then
    ctl.noteOff(lastNote)
  end

  ctl.noteOn(newNote, arp.velocity)
  arp.playedNotes[#arp.playedNotes + 1] = newNote

  arp.index = arp.index + 1
  if arp.index > #arp.notes then
    arp.index = 1
  end
end)
```

---

## Error Handling

All callbacks are safely wrapped. Errors are printed but never crash the synth engine.

---

This scripting system enables expressive, reactive, and algorithmic presets for advanced sound design in Mini-Moon.
