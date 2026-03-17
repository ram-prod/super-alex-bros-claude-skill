# Animation Patterns — Super Alex Bros

> This reference documents the ACTUAL Framer Motion patterns used in the project, plus recipes for new features. Match these patterns when adding animations — consistency is game feel.

## Table of Contents
1. [Project Animation Constants](#project-animation-constants)
2. [Existing Patterns (from codebase)](#existing-patterns-from-codebase)
3. [Recipes for New Features](#recipes-for-new-features)

---

## Project Animation Constants

These values are used consistently across the codebase. Stick to them:

| Constant | Value | Used For |
|----------|-------|----------|
| Spring stiffness | 150-300 | Entrances, emphasis |
| Spring damping | 12-15 | Natural settle |
| Character bob | y: [0, -6, 0], 2s | Idle animation |
| Stagger delay | 0.08-0.12s | List items |
| View transition | 0.3s fade | App.jsx AnimatePresence |
| Text stroke | 2px rgba(0,0,0,0.7) | TV readability |

---

## Existing Patterns (from codebase)

### View Transitions (App.jsx)
```jsx
<AnimatePresence mode="wait">
  <motion.div
    key={gamePhase}
    initial={{ opacity: 0 }}
    animate={{ opacity: 1 }}
    exit={{ opacity: 0 }}
    transition={{ duration: 0.3 }}
  >
    {currentView}
  </motion.div>
</AnimatePresence>
```

### Character Entrance (BattleView)
```jsx
// Player 1 enters from left
initial={{ x: '-100vw', opacity: 0 }}
animate={{ x: 0, opacity: 1 }}
transition={{ type: 'tween', ease: 'easeOut', duration: 0.3 }}

// Player 2 enters from right
initial={{ x: '100vw', opacity: 0 }}
animate={{ x: 0, opacity: 1 }}
```

### Character Idle Bob (BattleView)
```jsx
animate={{ y: [0, -6, 0] }}
transition={{ repeat: Infinity, duration: 2, ease: 'easeInOut' }}
```

### FIGHT! Title Slam (BattleView)
```jsx
initial={{ scale: 5, opacity: 0, rotate: -10 }}
animate={{ scale: 1, opacity: 1, rotate: 0 }}
transition={{ type: 'tween', ease: 'easeOut', duration: 0.2 }}
// Yellow flash background behind it
```

### Hit Flash (BattleView)
```jsx
animate={{
  filter: [
    'brightness(1) saturate(1) hue-rotate(0deg)',
    'brightness(3) saturate(0) hue-rotate(0deg)',
    'brightness(1.5) saturate(1.2) hue-rotate(-30deg)',
    'brightness(1) saturate(1) hue-rotate(0deg)'
  ]
}}
transition={{ duration: 0.6, times: [0, 0.2, 0.5, 1] }}
```

### Projectile Animation (BattleView)
```jsx
// 🍺 emoji flies in an arc
animate={{
  left: ['start', 'end'],    // Horizontal travel
  y: [0, -250, 0],           // Parabolic arc
  rotate: [0, 360, 720],     // Double spin
}}
transition={{ duration: 1.5, ease: 'linear' }}
```

### Particle Explosion (BattleView - HitExplosion)
```jsx
// 12 particles with random offsets
const particles = Array.from({ length: 12 }, () => ({
  x: (Math.random() - 0.5) * 200,
  y: (Math.random() - 0.5) * 200,
  delay: Math.random() * 0.1,
}));

// Each particle
animate={{ x: offset, y: offset, opacity: 0, scale: 0 }}
transition={{ duration: 0.6, delay: particle.delay }}
```

### Damage Number Pop (BattleView)
```jsx
<motion.span
  key={damage}  // Key change triggers remount → animation
  initial={{ scale: 1.6, y: -8 }}
  animate={{ scale: 1, y: 0 }}
  transition={{ type: 'spring', stiffness: 300, damping: 15 }}
>
  {damage}%
</motion.span>
```

### VS Screen (VsScreenView)
```jsx
// VS text
initial={{ scale: 5, opacity: 0 }}
animate={{ scale: 1, opacity: 1 }}
transition={{ type: 'spring', stiffness: 150, damping: 12 }}

// Players slide in from sides
initial={{ x: '-100vw' }} // or '100vw'
animate={{ x: 0 }}
transition={{ type: 'spring' }}

// Fighter pulse
animate={{ scale: [1, 1.1, 1] }}
transition={{ repeat: Infinity, duration: 2 }}
```

### Splash Screen (SplashView)
```jsx
// Title words stagger in
initial={{ opacity: 0, y: 30 }}
animate={{ opacity: 1, y: 0 }}
transition={{ delay: index * 0.5 }}

// Menu items from alternating sides
initial={{ opacity: 0, x: isEven ? -60 : 60 }}
animate={{ opacity: 1, x: 0 }}

// Button glow
animate={{ borderColor: [...], boxShadow: [...] }}
transition={{ repeat: Infinity, duration: 2 }}
```

### Map Card Hover (MapSelectView)
```jsx
whileHover={{ scale: 1.04, y: -6 }}
transition={{ type: 'spring', stiffness: 300, damping: 20 }}
// Image inside: group-hover:scale-115 (Tailwind)
```

### Staggered List (MapSelectView, RosterView)
```jsx
// Container doesn't need variants — just delay each child
{items.map((item, i) => (
  <motion.div
    key={item.id}
    initial={{ opacity: 0, y: 20 }}
    animate={{ opacity: 1, y: 0 }}
    transition={{ delay: i * 0.08 }}
  />
))}
```

---

## Recipes for New Features

### Announcer Banner (for round intros)
```jsx
const AnnouncerBanner = ({ text, onComplete }) => {
  useEffect(() => {
    const timer = setTimeout(onComplete, 2500);
    return () => clearTimeout(timer);
  }, [onComplete]);

  return (
    <motion.div
      className="fixed inset-0 flex items-center justify-center z-50 bg-black/80"
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      exit={{ opacity: 0 }}
    >
      <motion.h1
        className="text-7xl md:text-9xl font-black text-white uppercase tracking-wider"
        style={{ WebkitTextStroke: '3px rgba(0,0,0,0.7)' }}
        initial={{ scale: 5, opacity: 0, rotate: -10 }}
        animate={{ scale: 1, opacity: 1, rotate: 0 }}
        transition={{ type: 'spring', damping: 8, stiffness: 150 }}
      >
        {text}
      </motion.h1>
    </motion.div>
  );
};
```

### Screen Shake (for impacts)
```jsx
const useScreenShake = () => {
  const [offset, setOffset] = useState({ x: 0, y: 0 });

  const shake = useCallback((intensity = 5, duration = 300) => {
    const start = Date.now();
    const tick = () => {
      const elapsed = Date.now() - start;
      if (elapsed > duration) { setOffset({ x: 0, y: 0 }); return; }
      const decay = 1 - elapsed / duration;
      setOffset({
        x: (Math.random() - 0.5) * intensity * decay * 2,
        y: (Math.random() - 0.5) * intensity * decay * 2,
      });
      requestAnimationFrame(tick);
    };
    requestAnimationFrame(tick);
  }, []);

  return { offset, shake };
};

// Wrap game container
<motion.div animate={{ x: offset.x, y: offset.y }}>
```

### Score Counter (animated count-up)
```jsx
function useCountUp(target, duration = 0.8) {
  const [display, setDisplay] = useState(target);
  const prev = useRef(target);

  useEffect(() => {
    const start = prev.current;
    const diff = target - start;
    const startTime = Date.now();
    const tick = () => {
      const elapsed = (Date.now() - startTime) / (duration * 1000);
      if (elapsed >= 1) { setDisplay(target); prev.current = target; return; }
      const progress = 1 - Math.pow(1 - elapsed, 3); // ease-out
      setDisplay(Math.round(start + diff * progress));
      requestAnimationFrame(tick);
    };
    requestAnimationFrame(tick);
  }, [target, duration]);

  return display;
}
```

### Timer with Urgency (for timed rounds)
```jsx
const GameTimer = ({ seconds }) => (
  <motion.div
    animate={seconds <= 5 ? {
      scale: [1, 1.15, 1],
      color: ['#FF4444', '#FF0000', '#FF4444'],
    } : {}}
    transition={seconds <= 5 ? { repeat: Infinity, duration: 0.8 } : {}}
    className="text-6xl font-mono font-black"
    style={{ WebkitTextStroke: '2px rgba(0,0,0,0.7)' }}
  >
    {seconds}
  </motion.div>
);
```

### Smash-Style Screen Wipe (alternative to fade)
```jsx
<motion.div
  initial={{ clipPath: 'polygon(0 0, 0 0, 0 100%, 0 100%)' }}
  animate={{ clipPath: 'polygon(0 0, 100% 0, 100% 100%, 0 100%)' }}
  exit={{ clipPath: 'polygon(100% 0, 100% 0, 100% 100%, 100% 100%)' }}
  transition={{ duration: 0.4, ease: [0.76, 0, 0.24, 1] }}
>
```

---

> When creating new animations, always match the project's spring constants and timing. Test on a large screen — what looks smooth on a laptop may feel different on a TV at party distance.
