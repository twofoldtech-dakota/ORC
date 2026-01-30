# Stripe.com UI Patterns

## What Makes Stripe Exceptional

### Pixel-Perfect Alignment
- Everything aligns to grid
- Consistent spacing throughout
- Attention to optical alignment
- Perfect typography baseline

### Dashboard Patterns
- Information density that scales
- Clear data hierarchy
- Subtle depth through shadows
- Professional color palette

### Form Design
- Clear labels and helper text
- Smooth validation feedback
- Input states clearly defined
- Error recovery guidance

### Techniques to Adopt
1. Multi-column dashboard layouts
2. Subtle elevation through shadows
3. Data tables with clear hierarchy
4. Form validation with helpful messaging
5. Professional, not corporate aesthetic

## Example Implementation

```tsx
// Stripe-Style Form Input
<div className="space-y-1">
  <label
    htmlFor="email"
    className="block text-sm font-medium text-neutral-700 dark:text-neutral-300"
  >
    Email address
  </label>
  <input
    id="email"
    type="email"
    className={cn(
      "block w-full px-3 py-2",
      "border border-neutral-300 dark:border-neutral-700",
      "rounded-md shadow-sm",
      "focus:ring-2 focus:ring-primary-500 focus:border-transparent",
      "placeholder:text-neutral-400",
      "transition-all duration-150"
    )}
    placeholder="you@example.com"
  />
  <p className="text-xs text-neutral-500">
    We'll never share your email with anyone else.
  </p>
</div>

// Dashboard Card with Subtle Shadow
<div className={cn(
  "bg-white dark:bg-neutral-900",
  "border border-neutral-200 dark:border-neutral-800",
  "rounded-lg shadow-sm",
  "hover:shadow-md transition-shadow duration-200",
  "p-6"
)}>
  {/* Card content */}
</div>
```

## Key Principles
- Pixel-perfect alignment
- Professional aesthetic
- Clear data hierarchy
- Helpful form validation
