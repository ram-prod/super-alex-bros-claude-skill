# Content Systems — Super Alex Bros

> This reference covers the game's content data structures, how to expand them, and the scoring/progression design. Read this when adding content, creating new content types, or designing scoring mechanics.

## Table of Contents
1. [Current Content State](#current-content-state)
2. [gamedata.json Structure](#gamedatajson-structure)
3. [Expanding Content](#expanding-content)
4. [Scoring & Damage](#scoring--damage)
5. [Content Templates](#content-templates)
6. [Game Energy Curve](#game-energy-curve)

---

## Current Content State

**File:** `src/data/gamedata.json`

The game currently has **5 placeholder content items**. This is the primary expansion area. The infrastructure for loading and cycling through content exists — it just needs more entries.

**Question selection logic** (in `useGameStore.js`):
- Random selection from questions NOT in `askedQuestionIds`
- When all questions asked, pool resets (wraps around)
- No category filtering yet — all questions in one pool

---

## gamedata.json Structure

```json
{
  "characters": [
    { "id": "ruggero", "name": "Ruggero", "emoji": "🔥" },
    { "id": "alexander", "name": "Alexander", "emoji": "👑" }
    // ... 11 total
  ],
  "maps": [
    { "id": "amsterdam", "name": "Amsterdam Hotelschool", "image": "amsterdam.webp" },
    { "id": "villa", "name": "The Bachelor Villa", "image": "villa.webp", "isSecret": true }
    // ... 9 total (8 public + 1 secret)
  ],
  "content": [
    { "id": 1, "type": "trivia", "question": "What year was original Smash Bros released?", "answer": "1999" },
    { "id": 3, "type": "challenge", "question": "First to press button wins!", "answer": null }
    // ... 5 total (placeholder set)
  ]
}
```

**Characters are real people** — the bachelor's friend group:
Ruggero, Koen, Matthew, Martin, Robin, Frederik, Vincent, Devan, Gereon, Noah, and Alexander (the bachelor).

**Maps are real locations** meaningful to the group — Amsterdam, Düsseldorf, Berlin, Luxembourg, Barcelona, Lisbon, Valencia, New York, and The Bachelor Villa.

---

## Expanding Content

### Adding Trivia Questions

The current schema is minimal. For expansion, consider enriching the schema:

```json
{
  "id": 6,
  "type": "trivia",
  "category": "bachelor-knowledge",
  "question": "What was Alexander's first job?",
  "answer": "Bartender at Hotel School",
  "options": ["Bartender at Hotel School", "Waiter at Pizza Hut", "Lifeguard", "DJ"],
  "difficulty": "medium",
  "points": 100
}
```

**Important:** The BattleView currently renders questions as plain text with a "Reveal Answer" button. If you add `options` for multiple choice, you'll need to update BattleView to render option cards. Keep backward compatibility — questions without `options` should still work as open-ended.

**Suggested categories:**
- `bachelor-knowledge` — "How well do you know Alexander?"
- `group-history` — shared memories, trips, inside jokes
- `general` — pop culture, sports, random fun facts
- `embarrassing` — stories the bachelor wishes you'd forget
- `couples` — about Alexander and his partner

### Adding Challenges

Challenges currently have `answer: null`. Expand with more metadata:

```json
{
  "id": 10,
  "type": "challenge",
  "subtype": "physical",
  "question": "Arm wrestling! Loser takes a sip.",
  "answer": null,
  "duration": 30,
  "intensity": "medium",
  "bachelorVariant": "Alexander must use his non-dominant hand"
}
```

**Challenge subtypes to consider:**
- `physical` — arm wrestling, balance, speed
- `social` — call someone, tell a story, impersonate
- `drinking` — waterfall, categories, dare-or-drink
- `creative` — draw, act, sing
- `team` — group coordination

### Adding New Content Types

Beyond trivia and challenge, you could add:
- `"type": "vote"` — group votes on something (most likely to...)
- `"type": "hot-seat"` — bachelor answers, others bet on the answer
- `"type": "rapid-fire"` — speed round, many quick questions

Each new type needs corresponding UI in BattleView (or a new phase/view if it's significantly different from the battle format).

---

## Scoring & Damage

**Current system:**
- Damage-based: 0% → 100% → 200% = KO
- Binary outcome per question: one player "wins" (quiz master decides)
- Winner's opponent takes +100% damage
- No points, streaks, or bonuses yet

**Expansion opportunities:**
- **Points alongside damage** — track cumulative points even though damage decides matches
- **Speed bonuses** — if timer added, faster answers = bonus points
- **Streak tracking** — consecutive wins earn multiplier
- **Bachelor bonus** — Alexander earns bonus points in certain categories
- **Audience involvement** — spectator-visible leaderboard between matches

**Player stats tracked:**
```js
player.wins      // matches won
player.losses    // matches lost
player.isEliminated  // knocked out of tournament
```

These could be expanded with `triviaCorrect`, `challengesWon`, `damageDealt`, etc.

---

## Content Templates

### Template: Trivia Set (copy and fill in)

```json
[
  { "id": 100, "type": "trivia", "question": "YOUR QUESTION?", "answer": "CORRECT ANSWER" },
  { "id": 101, "type": "trivia", "question": "YOUR QUESTION?", "answer": "CORRECT ANSWER" },
  { "id": 102, "type": "trivia", "question": "YOUR QUESTION?", "answer": "CORRECT ANSWER" }
]
```

### Template: Challenge Set

```json
[
  { "id": 200, "type": "challenge", "question": "CHALLENGE DESCRIPTION", "answer": null },
  { "id": 201, "type": "challenge", "question": "CHALLENGE DESCRIPTION", "answer": null }
]
```

### Template: Announcer Lines (for future announcer system)

```json
{
  "roundIntros": {
    "trivia": ["🧠 TRIVIA TIME!", "TEST YOUR KNOWLEDGE!", "WHO'S THE SMARTEST?"],
    "challenge": ["⚡ CHALLENGE ROUND!", "PROVE YOURSELF!", "TIME TO GET PHYSICAL!"]
  },
  "reactions": {
    "correct": ["NICE ONE!", "TOO EASY!", "KNOWLEDGE IS POWER!"],
    "wrong": ["OUCH!", "NOT EVEN CLOSE!", "BRUTAL!"],
    "ko": ["K.O.!!!", "ELIMINATED!", "SEE YA LATER!"]
  }
}
```

---

## Game Energy Curve

Design the round sequence to maintain party energy:

```
Energy ▲
       │    ╭──╮          ╭────╮
       │   ╭╯  ╰╮  ╭──╮ ╭╯    ╰──╮  ╭──────╮
       │  ╭╯    ╰╮╭╯  ╰╮╯       ╰╮╭╯      ╰──╮
       │ ╭╯      ╰╯    ╰╯         ╰╯           ╰─ SIKE!
       │╭╯
       ╰┼────┼────┼────┼────┼────┼────┼────┼────►
        Prelims  Wildcards  QF    SF     Final  Reveal
```

The tournament structure naturally creates this arc:
- **Prelims**: Warmup, everyone plays, low stakes
- **Wildcards**: Surprise resurrection, hype moment
- **QF/SF**: Stakes rise, fewer players, more attention per match
- **Final**: Maximum tension
- **SIKE reveal**: The payoff — Alexander wins regardless, laughter erupts

Content should match: easy/fun questions in prelims, harder/more personal in later rounds.
