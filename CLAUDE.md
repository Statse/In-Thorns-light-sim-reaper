# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`light-sim-vst` is a live-show light simulator for 4x Cameo PixBar 650 CPro COB fixtures. The show MIDI file (`lights.mid`) triggers DMXIS scenes over MIDI, which drive real DMX lights. This plugin simulates those lights visually in Reaper without requiring the physical hardware.

## Plugin Format: JSFX

The plugin is a **JSFX** file (`light-sim.jsfx`) — Reaper's built-in plain-text scripting format. No compilation is needed.

### Installation (no build system)
1. Copy `light-sim.jsfx` to:
   ```
   C:\Users\<you>\AppData\Roaming\REAPER\Effects\
   ```
2. In Reaper, add a new FX on any track → search "Light Sim"
3. Route the DMXIS MIDI track output (or lights.mid playback) to this track

### Development workflow
- Edit `light-sim.jsfx` in any text editor
- In Reaper: right-click the plugin → "Edit" to open the built-in editor with live reload
- Changes apply immediately on save — no restart needed

## Architecture

### JSFX sections
| Section | Purpose |
|---------|---------|
| `@init` | Memory layout, state vars, placeholder color table |
| `@block` | Called every audio block — reads MIDI, updates fixture state |
| `@gfx` | Renders the UI (called on repaint) |

### MIDI protocol (from analysis of `lights.mid`)
- **ch 15**: Show page selector — each note = a different song/section
- **ch 16**: Scene trigger — note number selects a DMXIS preset (18 unique scenes, notes 0–17)
- **Velocities**: 127 = full intensity, 96 = reduced intensity section
- **Behavior**: last scene holds until next trigger (DMXIS default)

### Fixtures
4x Cameo PixBar 650 CPro COB, stage layout left→right:

| | Fixture 1 | Fixture 2 | Fixture 3 | Fixture 4 |
|-|-----------|-----------|-----------|-----------|
| Orientation | Horizontal | Vertical | Vertical | Horizontal |
| DMX start | 1 | 33 | 65 | 97 |
| Channels | 32 | 32 | 32 | 32 |

Each fixture has 6 COB sections. Within a fixture's 32 DMX channels:
- ch +0: master dimmer
- ch +2 to +25: pixel color data (24 channels, likely 6 sections × 4 RGBW)

### Memory layout in `@init`
```
mem[0..3]   = fix_r[0..3]        — current R per fixture
mem[4..7]   = fix_g[0..3]        — current G per fixture
mem[8..11]  = fix_b[0..3]        — current B per fixture
mem[100..]  = note_colors[...]   — placeholder RGB per scene note (18 × 3)
```

## Key Files

| File | Purpose |
|------|---------|
| `light-sim.jsfx` | The plugin — edit this |
| `lights.mid` | Show MIDI file (90 BPM, 77 min, 1836 scene triggers) |
| `PLAN.md` | Phased development plan with open questions |
| `resources/dmx-analysis.json` | Decoded MIDI timeline and fixture layout |
| `resources/set/.../*.rpp` | Reaper show session (DMXIS plugin embedded as base64 XML) |

## What's Needed for Phase 3 (Real Scene Colors)

The DMXIS show file (`.dps`) from the show computer contains the actual RGB values per scene. It lives at:
```
C:\Users\<username>\AppData\Roaming\db audioware\DMXIS\Shows\
```
Once added to `resources/`, the scene color table in `@init` can be replaced with real data decoded from it.
