# Animation References

## Core Animation Principles

### Timing Functions

#### Spring Physics (Preferred)
```tsx
// Framer Motion
const springConfig = {
  type: "spring",
  stiffness: 400,  // How tight the spring is
  damping: 17       // How much bounce/oscillation
};

// Use cases:
// - Interactive elements (buttons, cards)
// - Modal/dialog entrances
// - Drag interactions
```

#### Custom Cubic Bezier
```css
/* Ease-out (most common for UI) */
transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);

/* Bounce effect */
transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);

/* NEVER use linear for UI animations */
```

### Duration Guidelines

| Interaction Type | Duration | Timing Function |
|-----------------|----------|-----------------|
| Micro-interactions (hover, focus) | 150ms | Spring or ease-out |
| State changes (expand, collapse) | 200-250ms | Spring |
| Page transitions | 250-300ms | Ease-out |
| Loading states | 300ms+ | Spring with damping |

### Animation Patterns

#### 1. Hover Effects
```tsx
// Scale + Shadow (Linear-style)
<motion.button
  whileHover={{ scale: 1.02, boxShadow: "0 4px 12px rgba(0,0,0,0.1)" }}
  whileTap={{ scale: 0.98 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
>
  Click me
</motion.button>
```

#### 2. Page Transitions
```tsx
// Cross-fade with vertical shift
<AnimatePresence mode="wait">
  <motion.div
    key={pathname}
    initial={{ opacity: 0, y: 10 }}
    animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -10 }}
    transition={{ duration: 0.2, ease: "easeOut" }}
  >
    {children}
  </motion.div>
</AnimatePresence>
```

#### 3. Staggered List Animation
```tsx
const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: {
      staggerChildren: 0.05
    }
  }
};

const item = {
  hidden: { opacity: 0, y: 10 },
  show: { opacity: 1, y: 0 }
};

<motion.ul variants={container} initial="hidden" animate="show">
  {items.map((item) => (
    <motion.li key={item.id} variants={item}>
      {item.name}
    </motion.li>
  ))}
</motion.ul>
```

#### 4. Modal/Dialog Entrance
```tsx
const backdrop = {
  hidden: { opacity: 0 },
  visible: { opacity: 1 }
};

const modal = {
  hidden: { opacity: 0, scale: 0.95, y: 10 },
  visible: {
    opacity: 1,
    scale: 1,
    y: 0,
    transition: { type: "spring", stiffness: 300, damping: 20 }
  },
  exit: { opacity: 0, scale: 0.95, y: 10 }
};

<AnimatePresence>
  {isOpen && (
    <>
      <motion.div
        variants={backdrop}
        initial="hidden"
        animate="visible"
        exit="hidden"
        className="fixed inset-0 bg-black/50"
      />
      <motion.div
        variants={modal}
        initial="hidden"
        animate="visible"
        exit="exit"
        className="modal-content"
      >
        {children}
      </motion.div>
    </>
  )}
</AnimatePresence>
```

#### 5. Loading Skeleton
```tsx
// Pulse animation for skeleton screens
<div className="animate-pulse space-y-4">
  <div className="h-4 bg-neutral-200 dark:bg-neutral-800 rounded w-3/4" />
  <div className="h-4 bg-neutral-200 dark:bg-neutral-800 rounded w-1/2" />
</div>

// Custom pulse with smoother timing
@keyframes pulse-smooth {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

.animate-pulse-smooth {
  animation: pulse-smooth 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
```

## Accessibility

### Reduced Motion Support
```tsx
// Framer Motion
import { useReducedMotion } from "framer-motion";

function Component() {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      animate={{
        x: shouldReduceMotion ? 0 : 100,
        transition: { duration: shouldReduceMotion ? 0 : 0.3 }
      }}
    >
      {children}
    </motion.div>
  );
}

// CSS
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Anti-Patterns

### ❌ Never Do These
1. **Linear easing for UI animations**
   ```css
   /* BAD */
   transition: all 0.3s linear;

   /* GOOD */
   transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
   ```

2. **Uniform stagger delays**
   ```tsx
   /* BAD */
   transition={{ delay: index * 0.1 }}

   /* GOOD */
   Use staggerChildren in parent variant
   ```

3. **Long durations**
   ```css
   /* BAD - Too slow, feels sluggish */
   transition: all 0.5s;

   /* GOOD */
   transition: transform 0.15s, opacity 0.2s;
   ```

4. **No exit animations**
   ```tsx
   /* BAD - Abrupt exit */
   {isOpen && <Modal />}

   /* GOOD - Smooth exit */
   <AnimatePresence>
     {isOpen && <Modal />}
   </AnimatePresence>
   ```

5. **Animating layout-shifting properties without care**
   ```css
   /* BAD - Causes layout shifts */
   transition: width 0.3s;

   /* GOOD - Use transform instead */
   transition: transform 0.3s;
   transform: scaleX(1.2);
   ```

## Performance Tips

1. **Use `transform` and `opacity` - GPU accelerated**
2. **Avoid animating `width`, `height`, `top`, `left`**
3. **Use `will-change` sparingly**
4. **Prefer CSS animations for simple transitions**
5. **Use Framer Motion for complex orchestrations**
