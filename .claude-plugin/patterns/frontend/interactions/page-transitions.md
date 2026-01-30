# Page Transitions

## Core Principles

1. **Fast but Smooth**: 200-300ms, never longer
2. **Directional**: Indicate navigation direction
3. **Exit Animations**: Equal importance to enter
4. **Reduced Motion**: Respect user preferences

## Transition Patterns

### 1. Cross-Fade (Remotion-style)

Best for: Documentation, content-heavy pages

```tsx
import { AnimatePresence, motion } from "framer-motion";
import { useLocation } from "react-router-dom";

const PageTransition = ({ children }) => {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={{ opacity: 0 }}
        transition={{ duration: 0.2, ease: "easeOut" }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

### 2. Slide + Fade (Linear-style)

Best for: Multi-step forms, wizards, hierarchical navigation

```tsx
const SlideTransition = ({ children }) => {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait" initial={false}>
      <motion.div
        key={location.pathname}
        initial={{ opacity: 0, x: 20 }}
        animate={{ opacity: 1, x: 0 }}
        exit={{ opacity: 0, x: -20 }}
        transition={{
          type: "spring",
          stiffness: 300,
          damping: 30
        }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

### 3. Scale from Center

Best for: Modal-like pages, detail views

```tsx
const ScaleTransition = ({ children }) => {
  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={{ opacity: 0, scale: 0.95 }}
        animate={{ opacity: 1, scale: 1 }}
        exit={{ opacity: 0, scale: 0.95 }}
        transition={{
          duration: 0.2,
          ease: [0.4, 0, 0.2, 1]
        }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

### 4. Directional Slide (Based on Navigation)

Best for: Tab navigation, sequential content

```tsx
const DirectionalSlide = ({ children, direction = 1 }) => {
  return (
    <AnimatePresence mode="wait" custom={direction}>
      <motion.div
        key={location.pathname}
        custom={direction}
        initial={(direction) => ({
          opacity: 0,
          x: direction > 0 ? 50 : -50
        })}
        animate={{ opacity: 1, x: 0 }}
        exit={(direction) => ({
          opacity: 0,
          x: direction > 0 ? -50 : 50
        })}
        transition={{
          type: "spring",
          stiffness: 300,
          damping: 30
        }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

### 5. Staggered Content Entry

Best for: Dashboard pages, content-rich layouts

```tsx
const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: {
      staggerChildren: 0.05,
      delayChildren: 0.1
    }
  }
};

const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 }
};

const StaggeredPage = ({ children }) => {
  return (
    <motion.div
      variants={container}
      initial="hidden"
      animate="show"
      className="space-y-6"
    >
      {React.Children.map(children, (child) => (
        <motion.div variants={item}>
          {child}
        </motion.div>
      ))}
    </motion.div>
  );
};
```

## Advanced Patterns

### Shared Element Transition

```tsx
import { motion, useMotionValue, useTransform } from "framer-motion";

// List View
<motion.div layoutId={`card-${id}`}>
  <img src={thumbnail} />
</motion.div>

// Detail View (same layoutId creates shared element transition)
<motion.div layoutId={`card-${id}`}>
  <img src={fullImage} />
</motion.div>
```

### Progress Indicator During Transition

```tsx
const PageWithProgress = ({ children }) => {
  const [progress, setProgress] = useState(0);

  return (
    <>
      {/* Progress bar */}
      <motion.div
        className="fixed top-0 left-0 right-0 h-1 bg-primary-600 origin-left z-50"
        initial={{ scaleX: 0 }}
        animate={{ scaleX: progress / 100 }}
        transition={{ duration: 0.3 }}
      />

      <AnimatePresence mode="wait" onExitComplete={() => setProgress(100)}>
        <motion.div
          key={location.pathname}
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          onAnimationStart={() => setProgress(50)}
          onAnimationComplete={() => setTimeout(() => setProgress(0), 500)}
        >
          {children}
        </motion.div>
      </AnimatePresence>
    </>
  );
};
```

### Route-Based Transition Selection

```tsx
const getTransition = (from, to) => {
  // Forward navigation (list -> detail)
  if (to.includes('/detail')) {
    return { x: 20, opacity: 0 };
  }

  // Backward navigation (detail -> list)
  if (from.includes('/detail')) {
    return { x: -20, opacity: 0 };
  }

  // Default cross-fade
  return { opacity: 0 };
};

const SmartTransition = ({ children }) => {
  const location = useLocation();
  const prevLocation = useRef(location);

  const transition = getTransition(
    prevLocation.current.pathname,
    location.pathname
  );

  useEffect(() => {
    prevLocation.current = location;
  }, [location]);

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={transition}
        animate={{ x: 0, opacity: 1 }}
        exit={transition}
        transition={{ duration: 0.25, ease: "easeOut" }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

## Loading States

### Skeleton During Transition

```tsx
const PageWithSkeleton = ({ children, isLoading }) => {
  return (
    <AnimatePresence mode="wait">
      {isLoading ? (
        <motion.div
          key="skeleton"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
        >
          <PageSkeleton />
        </motion.div>
      ) : (
        <motion.div
          key="content"
          initial={{ opacity: 0, y: 10 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -10 }}
          transition={{ duration: 0.25 }}
        >
          {children}
        </motion.div>
      )}
    </AnimatePresence>
  );
};
```

## Accessibility

### Reduced Motion Support

```tsx
import { useReducedMotion } from "framer-motion";

const AccessibleTransition = ({ children }) => {
  const shouldReduceMotion = useReducedMotion();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={shouldReduceMotion ? {} : { opacity: 0, y: 10 }}
        animate={shouldReduceMotion ? {} : { opacity: 1, y: 0 }}
        exit={shouldReduceMotion ? {} : { opacity: 0, y: -10 }}
        transition={{ duration: shouldReduceMotion ? 0 : 0.2 }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
};
```

### Focus Management

```tsx
const PageTransition = ({ children }) => {
  const mainRef = useRef(null);

  return (
    <AnimatePresence
      mode="wait"
      onExitComplete={() => {
        // Focus main content after transition
        mainRef.current?.focus();
        // Scroll to top
        window.scrollTo(0, 0);
      }}
    >
      <motion.main
        ref={mainRef}
        tabIndex={-1}
        key={location.pathname}
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={{ opacity: 0 }}
      >
        {children}
      </motion.main>
    </AnimatePresence>
  );
};
```

## Performance Tips

1. **Use `mode="wait"`** to prevent layout shift
2. **Keep transitions under 300ms** for perceived performance
3. **Avoid animating expensive properties** during page transitions
4. **Use `layout` animations sparingly** on page changes
5. **Preload next page content** before transition starts

## Anti-Patterns

- ❌ Transitions longer than 500ms
- ❌ Different enter/exit durations (feels broken)
- ❌ Animating height on page change
- ❌ Complex animations on every route change
- ❌ No `mode="wait"` causing overlap
- ❌ Not handling reduced motion preference
