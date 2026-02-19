# light-sim-vst

A Reaper JSFX plugin that visually simulates the live lighting rig for **In Thorns** shows.

> **Heads up:** This is hardcoded for one specific setup — the In Thorns 2026 live show. It won't do anything useful for anyone else's rig without significant modification. That said, the code is open and free to take as a starting point for your own fixture simulator.

---

## What it does

Renders a real-time visual simulation of four **Cameo PixBar 650 CPro COB** fixtures as they respond to the show's MIDI lighting cues. Instead of running the physical DMX rig, you can see what the lights are doing directly inside Reaper — useful for rehearsal, programming review, or show prep without the hardware.

```
Stage layout (left → right):

 [  Fixture 1  ]   [ F2 ]   [ F3 ]   [  Fixture 4  ]
  Horizontal           Vertical  Vertical    Horizontal
```

Each fixture shows 6 COB sections with glow and beam spill effects that react in real time to the MIDI scene triggers.

---

## Requirements

- [Reaper](https://www.reaper.fm/) (any recent version)
- The In Thorns show MIDI file (`lights.mid`) or a live MIDI source using the same protocol
- No other dependencies — JSFX is built into Reaper

---

## Installation

1. Copy `light-sim.jsfx` into your Reaper effects folder:
   ```
   C:\Users\<you>\AppData\Roaming\REAPER\Effects\
   ```
2. Open Reaper and create a new track (or use the existing DMXIS MIDI track)
3. Add FX → search for **"Light Sim"** → add it
4. Open the FX window — the simulator UI appears immediately

---

## Usage with the show MIDI file

1. Import `lights.mid` as a MIDI item on a track
2. Route that track's MIDI output to the Light Sim track, or add Light Sim directly on the same track
3. Hit play — the fixtures react to every scene change in the file

The plugin is pass-through: all MIDI is forwarded unchanged, so it can sit in the FX chain alongside other plugins without affecting anything.

---

## MIDI protocol

The show uses two MIDI channels:

| Channel | Purpose | Notes |
|---------|---------|-------|
| ch 15 | Show page / song marker | Note number = song index |
| ch 16 | Scene trigger | Notes 0–17 = 18 unique lighting scenes |

Velocity 127 = full intensity section, velocity 96 = reduced intensity section.
The plugin holds the last scene state until the next trigger — matching DMXIS behaviour.

---

## Current state (Phase 1)

Scene colors are **placeholders** — each note number maps to a generic color (red, blue, white, etc.) rather than the real programmed look. The actual scene colors live in the DMXIS show file (`.dps`) which has not yet been decoded.

Once the `.dps` file is available, the `note_colors` table in the `@init` section of `light-sim.jsfx` can be replaced with the real per-scene, per-fixture RGB values.

---

## Extending / hacking this

The entire plugin is a single plain-text file. Open it in any editor or use Reaper's built-in JSFX editor (right-click the plugin → Edit). Changes hot-reload instantly.

Key areas to modify:

- **Scene colors** — `note_colors` table in `@init` (~line 30). Replace placeholders with real values once the DMXIS data is available.
- **Per-fixture colors** — currently all fixtures get the same color. In Phase 3, `fix_r[i]`, `fix_g[i]`, `fix_b[i]` will be set independently per fixture based on scene data.
- **Fixture geometry** — `H_W`, `H_H`, `V_W`, `V_H`, `segments` constants in `@gfx`.
- **COB count** — change `segments = 6` if the fixture turns out to have a different number of chips.

---

## License

Apache 2.0 — see [LICENSE](LICENSE).
Free to use, fork, and modify. If you build something with it, a mention of the original author is appreciated but not required.

---

*Built for the In Thorns 2026 live show.*
