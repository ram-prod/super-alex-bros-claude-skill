# Audio System — Super Alex Bros

> This reference covers the audio architecture, mix hierarchy, BGM engine, SFX catalog, and ducking principles. Read this when adding sounds, adjusting volumes, or working on music transitions.

## Table of Contents
1. [Audio Philosophy](#audio-philosophy)
2. [Mix Hierarchy](#mix-hierarchy)
3. [BGM Engine](#bgm-engine)
4. [SFX Engine](#sfx-engine)
5. [BGM Track Switching](#bgm-track-switching)
6. [Mute Controls](#mute-controls)
7. [Adding New Audio](#adding-new-audio)

---

## Audio Philosophy

Audio is 50% of the experience. A muted game loses half its soul. Every moment has a soundscape:
- **Tension** → BGM builds, SFX sparse
- **Impact** → BGM ducks or cuts, SFX punches through
- **Celebration** → Layered SFX, BGM triumphant
- **The SIKE** → Total silence → double SFX blast. The silence IS the setup.

---

## Mix Hierarchy

Volume balance relative to each other (professional target — not yet fully implemented):

| Category | Relative Level | Principle |
|---|---|---|
| **Voice/Announcer** | 0 dB (reference) | Always cuts through everything |
| **Impact SFX** | -3 to -6 dB | Feedback-critical: KO, first blood |
| **UI SFX** | -6 dB | Clicks, transitions — present but not dominant |
| **BGM** | -6 to -12 dB | Foundation mood, ducks for everything above |

### Ducking Principles
- KO moment → BGM fades out (already implemented via `setBgmState('faded')`)
- SIKE moment → everything silent, then double SFX blast
- Voice/announcer plays → BGM should duck -6dB (not yet implemented)
- Crossfade between BGM tracks: fade out → brief silence → fade in (not yet implemented, currently hard-switches)

---

## BGM Engine

**Location**: `App.jsx` — single `<audio>` element via `audioRef`, looped.

**Max volume**: 0.4

**State machine** (`bgmState` in Zustand store):
- `'paused'` — silent, mute UI hidden
- `'playing'` — fades to 0.4 (step +0.04/100ms ≈ 1s ramp)
- `'faded'` — fades to 0.0 (step -0.04/100ms)

Track switching via `setBgmState('playing', trackName)`.

| Track | File | Context |
|---|---|---|
| Theme | `theme.mp3` | Menu, overview, non-battle screens |
| Regular | `regular_game.mp3` | Standard battles |
| Final | `final_game.mp3` | Grand Final only |

---

## SFX Engine

**Location**: `useGameStore.js` — `playSFX(sfxId, vol)` creates a new `Audio` instance per call (allows overlap).

| SFX ID | Volume | Trigger |
|---|---|---|
| `click` | 1.0 | Default button click (global listener in App.jsx) |
| `click_special` | 0.8 | COMMENCE BATTLE button |
| `click_epic` | 1.0 | Major action buttons |
| `ready_to_start` | 1.0 | PRESS START |
| `announcer_roster` | 1.0 | RosterView mount (500ms delay) |
| `first_blood` | 1.0 | First hit per player (0%→100%, never on KO) |
| `ko_jingle` | 1.0 | 200% KO hit |
| `sike` | 0.7 | SIKE moment (simultaneous with smash) |
| `smash` | 1.0 | SIKE moment (simultaneous with sike) |

---

## BGM Track Switching

| Trigger | Track | How |
|---|---|---|
| PRESS START click (SplashView) | `theme.mp3` | `setBgmState('playing')` — starts BGM for first time |
| `startBattle()` (non-final) | `regular_game.mp3` | `setBgmState('playing', 'regular_game')` |
| `startBattle()` (isFinal) | `final_game.mp3` | `setBgmState('playing', 'final_game')` |
| KO hit (BattleView) | — | `setBgmState('faded')` — fades to silence |
| `nextMatch()` (back to overview) | `theme.mp3` | Hard-set `currentTrack: 'theme'`, `bgmState: 'playing'` |
| `resetGame()` | `theme.mp3` | Hard-set `currentTrack: 'theme'`, `bgmState: 'paused'` |

Note: BGM stays `paused` until first PRESS START. The mute UI is hidden while `bgmState === 'paused'`.

---

## Mute Controls

Bottom-right hoverable menu with independent toggles:
- `isBgmMuted` → 🎵/🔇
- `isSfxMuted` → 💥/🔕

---

## Adding New Audio

When adding new SFX or BGM:
1. Place file in `/public/assets/audio/{id}.mp3`
2. Decide the SFX tier (see `brand-and-design.md` Click SFX Tiers)
3. Add to the catalog table above
4. Consider mix hierarchy — what category does it fall in?
5. Test: does the new sound compete with or complement existing audio at this moment?

## Improvement Opportunities
- Implement proper mix hierarchy volumes (currently most SFX at 1.0)
- Add crossfade between BGM tracks instead of hard-switch
- Implement ducking when announcer/voice plays
- Audio sprite sheet for UI SFX (reduce HTTP requests)
- Preload critical SFX on game start (avoid first-play latency)
- Consider Web Audio API for precise timing and effects
