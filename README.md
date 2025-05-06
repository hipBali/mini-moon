# Mini-Moon: Modular Polyphonic Synth Engine with Lua Scripting

Mini-Moon is a lightweight, modular software synthesizer engine designed with flexibility, real-time sound synthesis, and scripting in mind. Inspired by the classic MiniMoog and named as a nod to the Lua scripting language ("lua" means "moon" in Portuguese), Mini-Moon allows users to create and manipulate rich soundscapes through code.

This project is aimed at hobbyists and tinkerers who want to dive deeper into custom sound design, from building simple subtractive patches to complex modulated instruments controlled via code.

---

## Features

* Polyphonic subtractive synthesizer core
* Modular signal chain: oscillators, filters, LFOs, effects, envelope, master
* Preset system supports JSON, Lua or hybrid formats
* Built-in Lua scripting support for MIDI callbacks and modulation logic
* Real-time parameter control via Lua (live `set()` and `update()`)
* Cross-platform C++ engine (Windows, Linux; macOS WIP)
* Tiny footprint: designed for Pi-class hardware

---

## Getting Started

See the following docs for details:

* **[Preset Format Specification](doc/Preset-Format.md)** – how to define modules, parameters, and patch layout
* **[Scripting Reference](doc/Lua_Scripting_Guide.md)** – Lua API for MIDI input, note events, controller handling, and custom logic
* **[Modulation Matrix](doc/Modulation.md)** – callback-based LFO routing and creative modulation ideas
* **[Example Preset](doc/Example-Preset.md)** – annotated dual LFO patch with dynamic tempo sync
* **[Preset Editor](tools/Preset-Editor.md)** – visual web-based preset designer tool

---

## Build & Platform Notes

### Windows (MSYS2 + UCRT toolchain)

* Clean static builds via `mingw-w64-ucrt-x86_64-gcc`
* Requires MSYS2 + UCRT64 shell and Lua (install with `pacman -S mingw-w64-ucrt-x86_64-lua`)
* No DLL runtime dependencies
* See `Makefile.win`

### Linux (Ubuntu/Debian)

* Uses ALSA and PulseAudio backends
* Requires `build-essential`, `pkg-config`, `liblua5.4-dev`
* See `Makefile.linux`

### macOS

* Build with Homebrew (`brew install lua@5.4 pkg-config`)
* Uses CoreAudio/CoreMIDI backend by default
* See `Makefile.macos`

### Raspberry Pi

* Primary embedded target
* See 'linux' section

---

## License

Unlicense — use, modify, embed freely.

If you build something cool with it, let us know!
