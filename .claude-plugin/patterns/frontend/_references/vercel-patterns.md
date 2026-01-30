# Vercel.com UI Patterns

## What Makes Vercel Exceptional

### Information Hierarchy
- Bold, clear headings
- Information density without clutter
- White space as a design element
- Clear visual grouping

### Dark Mode Excellence
- True dark (not just gray)
- Subtle gradients for depth
- Glow effects instead of shadows
- High contrast text

### Performance Focus
- Minimal, purposeful animations
- Fast page loads as a feature
- Skeleton states that match content
- Optimistic UI updates

### Techniques to Adopt
1. Bold typography with clear hierarchy
2. Gradient borders and glows for dark mode
3. Information-dense dashboards that breathe
4. Skeleton states that match final layout
5. Performance metrics as UI elements

## Example Implementation

```tsx
// Dark Mode Gradient Border
className={cn(
  "border border-transparent",
  "dark:bg-gradient-to-r dark:from-neutral-800 dark:to-neutral-900",
  "dark:bg-clip-padding dark:border-neutral-700"
)}

// Skeleton Loader that Matches Content
<div className="space-y-4">
  {/* Header skeleton */}
  <div className="h-8 w-64 bg-neutral-200 dark:bg-neutral-800 rounded animate-pulse" />

  {/* Content skeleton - matches final layout */}
  <div className="grid grid-cols-3 gap-4">
    {[1, 2, 3].map((i) => (
      <div key={i} className="h-32 bg-neutral-200 dark:bg-neutral-800 rounded animate-pulse" />
    ))}
  </div>
</div>
```

## Key Principles
- Bold typography choices
- Performance as a feature
- Minimal but not boring
- Dark mode done right
