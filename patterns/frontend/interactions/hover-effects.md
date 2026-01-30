# Hover Effects

## Core Principles

1. **Provide Immediate Feedback**: User should know element is interactive
2. **Use Spring Physics**: Natural, not robotic
3. **Subtle, Not Distracting**: Enhancement, not decoration
4. **Performance First**: GPU-accelerated properties only

## Effect Patterns

### 1. Scale + Shadow (Linear-style)

Best for: Buttons, cards, interactive elements

```tsx
<motion.div
  whileHover={{
    scale: 1.02,
    boxShadow: "0 4px 12px rgba(0,0,0,0.1)"
  }}
  whileTap={{ scale: 0.98 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
>
  {children}
</motion.div>
```

**CSS Alternative**:
```css
.interactive-card {
  transition: transform 150ms cubic-bezier(0.4, 0, 0.2, 1),
              box-shadow 150ms cubic-bezier(0.4, 0, 0.2, 1);
}

.interactive-card:hover {
  transform: scale(1.02);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.interactive-card:active {
  transform: scale(0.98);
}
```

### 2. Lift (Vertical Translation)

Best for: Cards, list items, panels

```tsx
<motion.div
  whileHover={{ y: -4, boxShadow: "0 12px 24px rgba(0,0,0,0.12)" }}
  transition={{ duration: 0.2 }}
>
  {children}
</motion.div>
```

**CSS Alternative**:
```css
.lift-card {
  transition: transform 200ms ease-out, box-shadow 200ms ease-out;
}

.lift-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px rgba(0,0,0,0.12);
}
```

### 3. Border Glow (Vercel-style)

Best for: Dark mode, feature cards, premium elements

```tsx
<div className={cn(
  "relative overflow-hidden rounded-lg",
  "before:absolute before:inset-0",
  "before:rounded-lg before:p-[1px]",
  "before:bg-gradient-to-r before:from-primary-500 before:to-primary-600",
  "before:opacity-0 hover:before:opacity-100",
  "before:transition-opacity before:duration-300"
)}>
  <div className="relative z-10 bg-neutral-900 rounded-lg p-6">
    {children}
  </div>
</div>
```

### 4. Background Shift

Best for: Navigation items, tabs, subtle interactions

```css
.nav-item {
  position: relative;
  transition: color 150ms ease-out;
}

.nav-item::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,0.05);
  opacity: 0;
  transition: opacity 150ms ease-out;
}

.nav-item:hover::before {
  opacity: 1;
}

/* Dark mode variant */
.dark .nav-item::before {
  background: rgba(255,255,255,0.05);
}
```

### 5. Underline Expansion

Best for: Links, navigation, minimal interfaces

```css
.link {
  position: relative;
  text-decoration: none;
  color: inherit;
}

.link::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 0;
  height: 2px;
  background: currentColor;
  transition: width 200ms cubic-bezier(0.4, 0, 0.2, 1);
}

.link:hover::after {
  width: 100%;
}
```

### 6. Icon Bounce

Best for: Interactive icons, delete buttons, favorites

```tsx
<motion.button
  whileHover={{ scale: 1.1 }}
  whileTap={{ scale: 0.9 }}
  transition={{ type: "spring", stiffness: 400, damping: 10 }}
>
  <HeartIcon />
</motion.button>
```

### 7. Shimmer Effect

Best for: Loading states, preview cards, premium features

```css
@keyframes shimmer {
  0% {
    background-position: -1000px 0;
  }
  100% {
    background-position: 1000px 0;
  }
}

.shimmer {
  background: linear-gradient(
    90deg,
    rgba(255,255,255,0) 0%,
    rgba(255,255,255,0.2) 20%,
    rgba(255,255,255,0.5) 60%,
    rgba(255,255,255,0)
  );
  background-size: 1000px 100%;
  animation: shimmer 2s infinite;
}

.card:hover .shimmer {
  animation-play-state: running;
}
```

## Combining Effects

### Card with Multiple Hover States

```tsx
const Card = ({ children }) => (
  <motion.div
    className="relative rounded-lg border border-neutral-200 dark:border-neutral-800 p-6"
    whileHover="hover"
    initial="initial"
  >
    {/* Background glow */}
    <motion.div
      className="absolute inset-0 rounded-lg bg-primary-500/10"
      variants={{
        initial: { opacity: 0 },
        hover: { opacity: 1 }
      }}
      transition={{ duration: 0.2 }}
    />

    {/* Content with lift */}
    <motion.div
      className="relative z-10"
      variants={{
        initial: { y: 0 },
        hover: { y: -2 }
      }}
      transition={{ type: "spring", stiffness: 400, damping: 17 }}
    >
      {children}
    </motion.div>

    {/* Icon animation */}
    <motion.div
      className="absolute top-4 right-4"
      variants={{
        initial: { rotate: 0 },
        hover: { rotate: 15 }
      }}
    >
      <ArrowUpRight className="h-5 w-5" />
    </motion.div>
  </motion.div>
);
```

## Accessibility Considerations

### Reduced Motion Support

```tsx
import { useReducedMotion } from "framer-motion";

const Component = () => {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      whileHover={shouldReduceMotion ? {} : { scale: 1.05 }}
      transition={{ duration: shouldReduceMotion ? 0 : 0.2 }}
    >
      {children}
    </motion.div>
  );
};
```

### Focus State Matching Hover

```css
/* Ensure focus state is visible and matches hover */
.interactive {
  transition: all 150ms ease-out;
}

.interactive:hover,
.interactive:focus-visible {
  transform: scale(1.02);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.interactive:focus-visible {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
}
```

## Performance Tips

1. **Use `transform` and `opacity`** - GPU accelerated
2. **Avoid animating `width`, `height`, `top`, `left`** - causes reflow
3. **Use `will-change` sparingly** - only when needed
4. **Debounce complex hover effects** on long lists

## Anti-Patterns

- ❌ Hover effect that only changes opacity
- ❌ No transition (instant change)
- ❌ Linear timing function
- ❌ Hover effects on mobile (no hover state)
- ❌ Animations over 300ms for micro-interactions
- ❌ Multiple conflicting animations
