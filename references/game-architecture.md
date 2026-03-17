# Game Architecture Patterns — Super Alex Bros

> This reference covers the ACTUAL architecture of the project. Read this when making architectural decisions, adding new systems, or understanding how things connect.

## Table of Contents
1. [Zustand Store Architecture](#zustand-store-architecture)
2. [Game Phase Flow](#game-phase-flow)
3. [Tournament Bracket System](#tournament-bracket-system)
4. [Battle System](#battle-system)
5. [VIP/Bachelor System](#vipbachelor-system)
6. [Audio System](#audio-system)
7. [Component Conventions](#component-conventions)
8. [Quiz Master Controls](#quiz-master-controls)
9. [Adding New Game Phases](#adding-new-game-phases)

---

## Zustand Store Architecture

**File:** `src/store/useGameStore.js` (~634 lines)

The entire game is driven by a single Zustand store with `persist` middleware. All game logic lives here — there are no separate hooks, contexts, or utility files.

```jsx
// Pattern: All components access state the same way
const { gamePhase, players, setGamePhase } = useGameStore();
```

**Persistence:** Saved to localStorage key `super-alex-bros-storage`. Excludes transient audio state (`bgmState`, `isMusicPlaying`, `hasSeenIntro`).

**When adding new state or actions:**
- Add state fields to the initial state object in `create()`
- Add actions alongside existing ones (they use `get()` and `set()`)
- If state shouldn't persist across page reloads, add the field name to the `partialize` exclusion list
- Avoid creating separate stores — everything goes in `useGameStore`

---

## Game Phase Flow

The `gamePhase` string drives which view renders in `App.jsx`:

```
splash → roster_select → confirmation → vip_reveal/vip_roulette
  → tournament_overview → map_select → vs_screen → battle → victory
  → (back to tournament_overview for next match, or reset)
```

**App.jsx routing pattern:**
```jsx
// App.jsx uses a switch on gamePhase
switch (gamePhase) {
  case 'splash': return <SplashView />;
  case 'roster_select': return <RosterView />;
  // ... etc
}
```

**To add a new phase:**
1. Add the phase string to the flow in `useGameStore.js`
2. Create a `{Phase}View.jsx` component in `src/components/`
3. Add the case to the switch in `App.jsx`
4. Set appropriate BGM track if needed

---

## Tournament Bracket System

This is the most complex system in the game. Understand it before modifying anything tournament-related.

**BRACKET_CONFIG** (lookup table for 2-11 players):
```js
const BRACKET_CONFIG = {
  2:  { base: 'Final', prelims: 0, byes: 0, wildcards: 0 },
  3:  { base: 'SF', prelims: 1, byes: 1, wildcards: 0 },
  // ... up to 11 players
  11: { base: 'QF', prelims: 5, byes: 1, wildcards: 2 },
};
```

**Bracket generation flow (`generateTournament()`):**
1. Create Prelim matches with real player IDs
2. Build knockout rounds (QF → SF → Final) with **placeholder strings**
3. Resolve VIP placeholders immediately
4. Load first round into `pendingMatches`

**Placeholder system** — this is critical to understand:
- `W_Prelims_0` = "winner of prelim match 0"
- `WC_0` = "wildcard slot 0"
- `VIP_0` = "VIP player slot 0"
- `W_QF_0` = "winner of QF match 0"

These get resolved to real player IDs by `replacePlaceholder()` as matches complete. When a match finishes, `advanceTournament()` replaces the winner's placeholder in subsequent rounds.

**Round order:** `['Prelims', 'QF', 'SF', 'Final']`

---

## Battle System

**File:** `src/components/BattleView.jsx`

BattleView has its own internal state machine (`battleState`):
```
intro_arena → intro_p1 → intro_p2 → intro_fight → idle_question
  → action_throw → action_hit → (idle_question or ko_sequence)
```

**Damage model:**
- Each player starts at 0%
- +100% per lost question round
- 200% = KO (elimination)
- `awardDamage(loserId)` in the store handles the math

**Question flow:**
- Random unasked question from `gamedata.json` content array
- `askedQuestionIds` tracks what's been used (wraps around when exhausted)
- Two types: `"trivia"` (has answer) and `"challenge"` (answer: null)
- Quiz masters see "👁️ Reveal Answer" button, then click winner button

---

## VIP/Bachelor System

**Alexander** (character ID: `'alexander'`) is the bachelor. His special treatment:

1. **evaluateVipPhase()** runs after confirmation:
   - If bracket needs byes AND Alexander is selected → automatic VIP → `VipRevealView`
   - If bracket needs byes AND Alexander not selected → roulette → `VipRouletteView`
   - If no byes needed → skip VIP phase entirely

2. VIP gets automatic bye (skips early rounds)

3. **SIKE twist** (`VictoryView`): After the tournament winner is announced, a "SIKE!" flash plays and Alexander is revealed as the true champion. This is hardcoded — Alexander always wins in the end.

---

## Audio System

Managed in the Zustand store + `App.jsx` (where the `<audio>` element lives).

**BGM tracks** (in `public/assets/audio/`):
- `theme` — splash/menu
- `ready_to_start` — confirmation screen
- `regular_game` — normal matches
- `final_game` — final match
- `vs_screen` — VS screen

**SFX** (played via `playSFX()`):
- `click`, `click_epic`, `click_special` — UI feedback
- `first_blood` — first damage in a match
- `ko_jingle` — KO moment
- `smash` — hit impact
- `sike` — SIKE reveal
- `select`, `announcer_roster` — character select

**Pattern:** Views call `setCurrentTrack('track_name')` and `setBgmState('playing')` to change music. SFX via `playSFX('sfx_name')`.

---

## Component Conventions

- **Flat structure:** All views in `src/components/`, no subdirectories
- **Naming:** `{Phase}View.jsx` — matches the `gamePhase` string
- **No props:** Views pull everything from `useGameStore()` directly
- **Inline animations:** Framer Motion `motion.div` with inline animation props (not variant objects in separate files)
- **Tailwind for layout:** All styling via Tailwind utility classes, occasionally inline styles for dynamic values (shadows, text stroke)
- **Local state for UI concerns:** `useState` for animation phases, reveal states, UI toggles. Game state always in Zustand.

---

## Quiz Master Controls

**Current implementation** (BattleView.jsx):
- Two large colored buttons: "🏆 {Player} WINS!"
- Visible during `idle_question` battle state
- Click triggers `awardDamage()` → projectile → hit → next question or KO
- "👁️ Reveal Answer" button for trivia questions

**Pattern for new quiz master controls:**
- Use large, high-contrast buttons (min touch target: 48x48px, ideally larger)
- Place at bottom of screen or in a dedicated control bar
- Include keyboard shortcuts as enhancement (not requirement)
- Always provide visual feedback on press (scale/color change)

---

## Adding New Game Phases

To insert a new phase (e.g., a standalone minigame round between tournament matches):

1. **Store:** Add phase transition logic — where it comes from, where it goes
2. **Component:** Create `NewPhaseView.jsx` following existing conventions
3. **App.jsx:** Add the case to the switch statement
4. **Audio:** Set appropriate BGM track in the phase transition
5. **Animations:** Use AnimatePresence exit/enter — App.jsx handles the wrapper

The game phase system is linear but supports branching (see VIP reveal vs roulette). You can add conditional phases the same way.
