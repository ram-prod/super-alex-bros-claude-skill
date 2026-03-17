# Minigame & Round Framework — Super Alex Bros

> This reference covers how to add new round types and content to the game's tournament structure. Read this when creating new gameplay modes.

## Table of Contents
1. [How the Game Currently Works](#how-the-game-currently-works)
2. [Two Expansion Paths](#two-expansion-paths)
3. [Path A: New Content Within Battle](#path-a-new-content-within-battle)
4. [Path B: New Standalone Phases](#path-b-new-standalone-phases)
5. [Minigame Design Principles](#minigame-design-principles)
6. [Bachelor-Special Rounds](#bachelor-special-rounds)

---

## How the Game Currently Works

The game is a **single-elimination tournament bracket** where each match plays out in BattleView:

1. Quiz masters select a map → VS screen → Battle begins
2. A random question appears (trivia or challenge)
3. Quiz masters decide who wins → loser takes damage
4. 200% damage = KO → loser eliminated from tournament
5. Winner advances in bracket → next match

**Key insight:** The game doesn't have separate "round types" yet. Everything runs through the same BattleView with the same damage mechanic. The variety comes from the content (trivia vs challenge questions), not from different gameplay modes.

---

## Two Expansion Paths

### Path A: More Content Types in BattleView
Add new question types that render differently in BattleView but use the same damage system. Lower effort, maintains the tournament structure.

### Path B: Standalone Game Phases
Add entirely new views/phases that run between or instead of tournament matches. Higher effort, enables fundamentally different gameplay.

Both paths can coexist. Start with Path A for quick wins, evolve to Path B for major features.

---

## Path A: New Content Within Battle

The BattleView already switches display based on `activeQuestion.type`:
- `"trivia"` → shows "🧠 TRIVIA TIME" with reveal button
- `"challenge"` → shows "⚡ MINIGAME CHALLENGE"

**To add a new content type:**

1. Add entries to `gamedata.json`:
```json
{ "id": 50, "type": "vote", "question": "Who is most likely to get lost in a foreign city?", "answer": null }
```

2. Update BattleView to handle the new type:
```jsx
{activeQuestion.type === 'vote' && (
  <div className="text-center">
    <h3 className="text-2xl text-purple-400">🗳️ GROUP VOTE</h3>
    <p className="text-xl mt-4">{activeQuestion.question}</p>
    {/* Quiz master decides who "wins" the vote */}
  </div>
)}
```

3. Quiz masters still use the same "Player WINS" buttons to award damage.

**Possible new content types:**
| Type | Display | How Winner is Decided |
|------|---------|----------------------|
| `vote` | Group votes on who fits the description | Quiz master picks based on group consensus |
| `hot-seat` | Bachelor answers, others predict | Quiz master awards to correct predictor |
| `rapid-fire` | Quick questions, 5-second timer | Quiz master picks fastest correct answer |
| `dare` | Player must perform a dare | Quiz master judges completion |
| `memory` | Photo/story from the past | Quiz master awards to best answer |

---

## Path B: New Standalone Phases

For gameplay that doesn't fit the 1v1 damage model (e.g., all-player minigames, team challenges).

**Step-by-step:**

1. **Define the phase** in `useGameStore.js`:
   - Add a new `gamePhase` value (e.g., `'minigame_reaction'`)
   - Add transition logic: where it enters from, where it exits to
   - Consider: does this replace a tournament match, or run between them?

2. **Create the view** (`src/components/MinigameReactionView.jsx`):
   - Follow existing conventions: no props, pull state from `useGameStore`
   - Include its own internal state machine for phases (intro → playing → results)
   - Call a store action to transition back to tournament when done

3. **Wire it up** in `App.jsx`:
   ```jsx
   case 'minigame_reaction': return <MinigameReactionView />;
   ```

4. **Handle scoring**: Since this is outside the damage system, decide:
   - Does it award bonus points (new stat)?
   - Does it affect tournament seeding?
   - Does it give the winner an advantage in the next match?
   - Or is it just for fun / drinking rules?

**Integration patterns:**
- **Between rounds:** After a match completes, before the next match loads, insert the minigame phase
- **Scheduled:** Every N matches, trigger a group minigame
- **Random:** Small chance of a minigame instead of a regular match
- **Bachelor special:** Triggered at specific tournament stages (e.g., before SF)

---

## Minigame Design Principles

### The 5-Second Rule
If you can't explain the minigame in 5 seconds, it's too complex for a party with drinks flowing. The quiz masters need to understand it instantly.

### Duration Guide
- **Quick (15-30s):** Reaction speed, buzzer, single dare
- **Medium (1-2 min):** Multi-question trivia, group vote, physical challenge
- **Long (2-4 min):** Complex minigame, team challenge
- **Never exceed 5 minutes** — energy dies

### The Quiz Master Test
Every minigame must answer: "How do the quiz masters control this?"
- Can they advance/skip at any time?
- Can they manually override scoring if needed?
- Can they pause if the room needs a moment?
- Are the controls obvious without instructions?

### The Spectator Test
Is it fun to watch? A minigame where one person stares at a screen while 9 others wait is a party killer. Either everyone participates, or the action is visible and entertaining for spectators.

### The Drunk Test
Will this still work when everyone's 4 beers in? Avoid:
- Precise timing requirements (reaction games are OK, rhythm games get sloppy)
- Complex multi-step instructions
- Small text or subtle visual cues
- Anything requiring quiet concentration

---

## Bachelor-Special Rounds

Alexander gets spotlight moments. Design principles:

### Celebratory, Not Cruel
Even embarrassing content should come from love. The quiz masters are the safety valve — they can skip anything that crosses a line.

### Interactive for Everyone
Don't isolate Alexander. Best bachelor moments involve the whole group:
- **Everyone bets** on Alexander's answer
- **Everyone votes** on bachelor superlatives
- **Everyone helps** Alexander complete a challenge
- **Everyone reacts** to embarrassing photo/story reveals

### Existing Bachelor Mechanics to Extend
- VIP system (automatic bye) — works, keep it
- SIKE twist (always wins) — the payoff, don't mess with it
- Character `alexander` has `emoji: "👑"` — use crown motif

### Ideas for Bachelor Rounds
- **Hot Seat:** "What would Alexander do?" — group predicts, bachelor reveals
- **Memory Lane:** Photos from different eras, group guesses the year/context
- **Confession Booth:** Anonymous embarrassing stories submitted by friends, bachelor guesses who wrote each
- **Superlatives:** "Most likely to..." votes with Alexander-specific categories
