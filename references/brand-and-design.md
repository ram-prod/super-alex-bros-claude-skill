# Brand & Design System — Super Alex Bros

> This reference covers the visual design system, CSS utilities, typography, color palette, button patterns, and SFX click tiers. Read this when creating new UI components, styling elements, or making layout decisions.

## Table of Contents
1. [CSS Utility Classes](#css-utility-classes)
2. [Typography System](#typography-system)
3. [Color System](#color-system)
4. [Button Architecture](#button-architecture)
5. [Click SFX Tiers](#click-sfx-tiers)
6. [Layout Patterns](#layout-patterns)
7. [Party-Distance Readability](#party-distance-readability)

---

## CSS Utility Classes

Defined in `src/index.css`:

```css
.text-smash           → font-black italic uppercase tracking-wider drop-shadow-lg
.panel-smash          → bg-black/90 backdrop-blur-md border-4 rounded-none skewX(-3deg)
.panel-smash-content  → skewX(3deg)  /* counter-skew for readable content */
```

These are the building blocks. The underlying principles matter more than the exact classes:
- **text-smash**: Aggressive, loud, fight-announcer energy
- **panel-smash**: Dark glass panels with angular, aggressive framing
- If these classes evolve, update this file and `index.css` together

---

## Typography System

| Tier | Current Classes | Principle | Usage |
|---|---|---|---|
| **Headline** | `.text-smash` | Screams at you. Announcer voice. | View titles, SIKE, champion name |
| **Lore** | `font-bold uppercase tracking-widest text-gray-300 drop-shadow-md` | Authoritative, mysterious. Tournament narrator. | Rules, VIP reveal, sublabels |
| **Body** | `text-white` | Clean, readable. Stays out of the way. | Regular content, descriptions |

**Principle**: Typography is hierarchical. Headlines dominate. Lore sets tone. Body informs. Never let body text compete with headlines for attention.

---

## Color System

| Role | Current Values | Principle |
|---|---|---|
| **Primary action** | `bg-red-600` / `bg-red-700` | Hot, urgent, "press me now" |
| **Secondary/special** | `bg-purple-600` / `bg-pink-600` | Mysterious, magical (wildcards, roulette) |
| **Accent/champion** | `border-yellow-400`, gold tones | Victory, prestige, the crown |
| **Neon glow** | `shadow-[0_0_20px_rgba(255,0,0,0.5)]` | Arcade cabinet energy |
| **Canvas** | `bg-black`, `bg-gray-900` | Deep darkness makes everything pop |

**Principle**: The palette is neon-on-dark. New colors are welcome if they serve a purpose — but they must feel like they belong in an arcade at midnight, not a corporate dashboard.

---

## Button Architecture

Nested skew pattern to prevent Framer Motion transform conflicts:

```jsx
<motion.div whileHover={{ scale: 1.05 }} whileTap={{ scale: 0.95 }}>
  <div style={{ transform: 'skewX(-10deg)' }} className="bg-red-600 px-8 py-4">
    <div style={{ transform: 'skewX(10deg)' }} className="text-smash text-white">
      BUTTON TEXT
    </div>
  </div>
</motion.div>
```

**Why this exists**: Framer Motion overwrites inline `transform` when animating. Separating the animation target (`motion.div`) from the visual skew element prevents this. The visual goal is: angled, aggressive, arcade-style buttons that respond to interaction.

**Critical**: Never put `whileHover` or `whileTap` on the same element that has inline CSS `transform`. See SKILL.md's Critical Framer Motion Rules for details.

---

## Click SFX Tiers

| Tier | Attribute | Principle |
|---|---|---|
| **Epic** | `data-sound="epic"` | Major commitment actions (start game, lock roster, pick map) |
| **Special** | `data-sound="special"` | Dramatic threshold moments (commence battle) |
| **Default** | (none) | Everything else — navigation, toggles, minor actions |

**Principle**: SFX tier communicates *weight of action* to the user. New buttons should get the tier that matches their importance, not always default.

---

## Layout Patterns

**Card grids**: `flex flex-wrap justify-center` with calc-based widths.
**Why**: Centers orphan items in the last row naturally. CSS grid doesn't do this without hacks.
**When to use grid**: If a layout genuinely needs named areas or complex spanning.

**Spacing conventions**:
- `p-6` standard panel padding, `p-8` for emphasis
- `gap-4` between related items
- `py-12` minimum view padding
- Thick borders (`border-4`) — the game is aggressive, not delicate

---

## Party-Distance Readability

Everything must be readable/visible from ~3 meters on a TV:
- Minimum text size for important labels: `text-lg` or larger
- Interactive elements: large tap targets (`px-8 py-4` minimum for buttons)
- Animations: visible at room scale, not just laptop scale
- Contrast: white/neon on black — high contrast always

---

## Extending the Design System

When adding new UI elements:
1. Check if existing classes/patterns cover it
2. If not, create new utilities that follow the same principles
3. Document new patterns in this file
4. Ask: "Does this look like it belongs on the same arcade cabinet?"
