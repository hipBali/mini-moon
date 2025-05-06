# Web-based Synth Preset Editor

This tool provides a browser-based, platform-independent interface for creating and editing synthesizer presets. Presets can be saved in JSON format, and converted into Lua-compatible structures for use in a synthesizer engine.

---

## ✨ Features

- Add and configure:
  - 🎚️ Oscillators (type, gain, detune, pan, etc.)
  - 🔁 LFOs (target assignment, waveform, sync, depth)
  - 🎛️ Filters (type, cutoff, resonance — order-sensitive)
  - 🎧 Effects (chained with drag-reorder and full parameter sets)
  - 🎚️ Master settings (gain, pan, tempo, pitch, glide)

- Live Lua code generation
- JSON export & import
- Clipboard copy of generated Lua
- Toggle between JSON and Lua view

---

## 📦 Files

- `Preset-Editor.html` — Self-contained UI (HTML + JS)
- Presets are stored as `.json` and convertible to Lua

---

## 🚀 How to Use

1. Open `Preset-Editor.html` in any modern browser.
2. Add synth components using the provided buttons.
3. Adjust parameters via sliders and dropdowns.
4. Use:
   - **Save as JSON** to export your preset
   - **Open JSON preset...** to import an existing one
   - **Toggle View** to switch between Lua and JSON display
   - **Copy to Clipboard** to use Lua directly in your engine

---

## 🔁 Lua Integration

### Example output:
```lua
modules = {
   oscillator = {
      { name = "OSC-1", type = OscillatorType.Sine, gain = 0.5 }
   },
   filter = {
      { name = "FLT-1", type = FilterType.LowPass, cutoff = 800 }
   },
   effect = {
      { name = "FX-1", type = EffectType.Reverb, mix = 0.5 }
   },
   master = {
      gain = 0.8, pan = 0, tempo = 120
   }
}
