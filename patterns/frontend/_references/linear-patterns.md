# Linear.app UI Patterns

## What Makes Linear Exceptional

### Interaction Polish
- Every interactive element has hover, focus, active states
- Spring physics on hover (scale 1.02, spring easing)
- Keyboard shortcuts visible and accessible
- Immediate feedback on every action

### Animation Approach
- Spring-based, not CSS linear
- Typical config: stiffness 400, damping 17
- Duration: 150-200ms for micro-interactions
- Exit animations match enter animations

### Spacing System
- 8px base grid
- Consistent rhythm throughout
- Generous whitespace in focused views
- Dense but readable in list views

### Dark Mode
- Not inverted - intentionally designed
- Subtle gradients for depth
- Reduced contrast shadows (or glow)
- Accent colors adjusted for dark backgrounds

### Techniques to Adopt
1. Spring hover animation: `scale(1.02)` with spring physics
2. Gradient borders on focus states
3. Keyboard shortcut hints that appear on focus
4. Subtle background shifts on interactive elements
5. Command palette pattern (Cmd+K)

## Example Implementation

```tsx
// Framer Motion Spring Configuration
const springConfig = {
  type: "spring",
  stiffness: 400,
  damping: 17
};

// Interactive Card Component
<motion.div
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.98 }}
  transition={springConfig}
>
  {/* Card content */}
</motion.div>
```

## Key Principles
- Keyboard-first design
- Natural, physics-based motion
- Intentional dark mode
- Micro-interactions everywhere
