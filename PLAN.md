# Light Sim VST - Project Plan

## Project Description
A plugin that visually simulates 4x Cameo PixBar 650 CPro COB fixtures responding to a MIDI-driven DMX show in real time. MIDI note triggers → scene lookup → visual fixture display.

## Setup Summary
- **OS:** Windows only
- **DAW:** Reaper
- **Fixtures:** 4x Cameo PixBar 650 CPro COB
- **Stage layout (stage left → right):** Horizontal | Vertical | Vertical | Horizontal
- **Visual style:** Start simple (colored COB sections), upgrade later

## MIDI File Analysis (lights.mid)
- Format: Type 1, 90 BPM, 960 ticks/beat
- Track 1 ("DMXIS MIDI"): Note-trigger protocol
  - MIDI channel 14 and 15 (0-indexed)
  - Notes 0–13+ observed, all at velocity 127
  - Note ON = scene active, Note OFF = scene off
  - **Channel 14 and 15 are likely two different "pages" or fixture groups in DMXIS**

---

## Technical Approach: JSFX (Reaper-native, no build system needed)

Instead of a full C++ VST (which requires JUCE, CMake, Visual Studio, long build cycles), we will use **JSFX** — Reaper's built-in scripting language. This is the fastest path to a working result:

- Plain text `.jsfx` files, no compilation
- Drop into Reaper's Effects folder and it works immediately
- Built-in MIDI receive API
- Built-in graphics API (`gfx_*` functions) for drawing the light display
- Easy to iterate and modify
- If cross-DAW use is ever needed, we port to JUCE as a second phase

JSFX files live at: `C:\Users\<you>\AppData\Roaming\REAPER\Effects\`

---

## Open Questions

### RESOLVED
- [x] OS: Windows only
- [x] DAW: Reaper
- [x] Visual style: Start simple, upgrade later
- [x] DMXIS access: Can access show computer

### MUST RESOLVE BEFORE PHASE 3 (Scene mapping)
- [ ] **Q1: DMXIS scene data** — On the show computer, open DMXIS and document:
  - What does each MIDI note (ch14 note 0, ch14 note 1, ch15 note 0...) look like on the lights?
  - Or: export the scene list from DMXIS (File → Export or screenshot each scene)
  - What we need per scene: for each of the 4 fixtures → R, G, B values of each COB section

- [ ] **Q2: Fixture DMX mode** — Check the PixBar 650 CPro COB manual for which DMX mode is selected on each fixture (there is usually a menu on the back of the fixture). Different modes = different channel counts. Also note the DMX start address of each fixture (also on the fixture display).

- [ ] **Q3: Fixture COB section count** — How many individually addressable COB sections does the PixBar 650 CPro COB have? (Likely 6 or 8 — check the manual or count the physical chips on the bar)

---

## Phases

### Phase 1 — JSFX Scaffold ✅ COMPLETE
**Goal:** Plugin opens in Reaper, draws 4 fixture representations, confirms MIDI is received.

Tasks:
- [x] Create `light-sim.jsfx` in repo root (copy to Reaper Effects folder to install)
- [x] Set up MIDI input handling (`@block` section, reads ch15 + ch16 notes)
- [x] Draw 4 fixture bars in correct orientation (H|V|V|H) with 6 COB segments each
- [x] On note trigger (ch16), all fixtures light up with placeholder color per note
- [x] Display active scene (note, channel, velocity) in header
- [x] Display active show page (ch15 note) in header
- [x] Beam spill effect per fixture orientation
- [x] Footer shows ch15/ch16 protocol info

**Installation:** Copy `light-sim.jsfx` to `C:\Users\<you>\AppData\Roaming\REAPER\Effects\`

Deliverable: Plugin loads in Reaper, all 4 fixtures light up when MIDI file plays.

### Phase 2 — DMXIS Data Collection
**Goal:** Know exactly what R,G,B each fixture shows for every note in the MIDI file.

Tasks:
- [ ] Open DMXIS on show computer, play through lights.mid
- [ ] For each unique note trigger, record which fixtures light up and what color
- [ ] Build a `scenes.json` or table mapping: `{channel, note} → [{fixture, rgb_per_section}]`
- [ ] Find and document: fixture start addresses, DMX mode, COB section count

### Phase 3 — Scene Mapping
**Goal:** Plugin correctly shows real fixture colors per scene.

Tasks:
- [ ] Embed scene data in JSFX plugin (or load from a sidecar file)
- [ ] Map note triggers to full fixture states (all 4 fixtures, all COB sections)
- [ ] Handle simultaneous notes (scene stacking rules in DMXIS — typically last note wins, or additive)
- [ ] Handle Note OFF correctly (does fixture go dark, or revert to previous scene?)

### Phase 4 — Visual Polish
**Goal:** Looks good enough to use as a reference during production.

Tasks:
- [ ] Per-COB-section color rendering (not just whole-bar color)
- [ ] Add glow/bloom effect around lit sections (simple additive circle)
- [ ] Show fixture labels (L1 H / L2 V / L3 V / L4 H)
- [ ] Add BPM/timeline display if useful
- [ ] Consider background stage diagram (black stage, bar mounting positions)

### Phase 5 — (Optional) JUCE Port
If cross-DAW use is needed or more advanced graphics are wanted, port the logic to a proper C++ VST3 using JUCE.

---

## Data Collection Template (for Phase 2)

When on the show computer, for each unique note event in lights.mid:

| MIDI Ch | Note | Fixture 1 (H) | Fixture 2 (V) | Fixture 3 (V) | Fixture 4 (H) |
|---------|------|---------------|---------------|---------------|---------------|
| 14      | 0    | ?             | ?             | ?             | ?             |
| 14      | 1    | ?             | ?             | ?             | ?             |
| 14      | 2    | ?             | ?             | ?             | ?             |
| 15      | 0    | ?             | ?             | ?             | ?             |
| 15      | 1    | ?             | ?             | ?             | ?             |
| 15      | 2    | ?             | ?             | ?             | ?             |
| 15      | 3    | ?             | ?             | ?             | ?             |
| 15      | 4    | ?             | ?             | ?             | ?             |
| 15      | 5    | ?             | ?             | ?             | ?             |
| 15      | 6    | ?             | ?             | ?             | ?             |
| 15      | 7    | ?             | ?             | ?             | ?             |
| 15      | 8    | ?             | ?             | ?             | ?             |
| 15      | 9    | ?             | ?             | ?             | ?             |
| 15      | 10   | ?             | ?             | ?             | ?             |
| 15      | 11   | ?             | ?             | ?             | ?             |
| 15      | 12   | ?             | ?             | ?             | ?             |
| 15      | 13   | ?             | ?             | ?             | ?             |

For each cell, record: which COB sections are on, and what color (R,G,B).

---

## Notes & Decisions Log
- MIDI uses DMXIS note-trigger protocol, not raw DMX-over-MIDI
- Two MIDI channels (14, 15) — likely two DMXIS pages or fixture groups
- All note velocities are 127 — velocity not used for dimming in this show file
- JSFX chosen for Phase 1 due to: zero build system, native Reaper support, fast iteration
- Scene data is the critical unknown — cannot fully implement Phase 3 without it
- "Horizontal" fixture = bar laid flat, "Vertical" = bar standing upright (to be confirmed)
