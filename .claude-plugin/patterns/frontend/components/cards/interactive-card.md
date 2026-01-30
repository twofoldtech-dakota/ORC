# Interactive Card Pattern

## Reference Sources
- **Linear**: Subtle hover elevation with spring animation
- **Stripe**: Clean borders, subtle shadows for depth
- **Vercel**: Minimal design with intentional spacing

## States

### Default
- Background: `white` / `neutral-900` (dark)
- Border: 1px `neutral-200` / `neutral-800` (dark)
- Shadow: `sm`
- Border-radius: `lg`

### Hover (if clickable)
- Border: `neutral-300` / `neutral-700` (dark)
- Shadow: `md`
- Transform: `translateY(-2px)` or `scale(1.01)`
- Transition: 150ms spring

### Focus (if clickable)
- Ring: 2px `primary-500`, 2px offset
- Maintain hover state

### Active/Pressed (if clickable)
- Transform: `translateY(0)` or `scale(0.99)`
- Shadow: `sm`

## Code Example (React + Tailwind)

```tsx
import { cn } from "@/lib/utils";
import { motion } from "framer-motion";

interface CardProps {
  children: React.ReactNode;
  clickable?: boolean;
  onClick?: () => void;
  className?: string;
}

const Card = ({ children, clickable = false, onClick, className }: CardProps) => {
  const Component = clickable ? motion.button : motion.div;

  return (
    <Component
      onClick={onClick}
      whileHover={clickable ? { y: -2, boxShadow: "0 4px 12px rgba(0,0,0,0.08)" } : {}}
      whileTap={clickable ? { y: 0, scale: 0.99 } : {}}
      transition={{ type: "spring", stiffness: 400, damping: 17 }}
      className={cn(
        // Base
        "bg-white dark:bg-neutral-900",
        "border border-neutral-200 dark:border-neutral-800",
        "rounded-lg shadow-sm",
        "p-6",

        // Clickable variant
        clickable && [
          "cursor-pointer",
          "hover:border-neutral-300 dark:hover:border-neutral-700",
          "focus:outline-none focus-visible:ring-2",
          "focus-visible:ring-primary-500 focus-visible:ring-offset-2",
          "transition-colors duration-150"
        ],

        className
      )}
    >
      {children}
    </Component>
  );
};

export default Card;
```

## Layout Patterns

### Information Card
```tsx
<Card>
  <div className="flex items-start justify-between">
    <div className="flex-1">
      <h3 className="text-lg font-semibold text-neutral-900 dark:text-neutral-100">
        Card Title
      </h3>
      <p className="mt-1 text-sm text-neutral-500 dark:text-neutral-400">
        Supporting text that provides additional context
      </p>
    </div>
    <Badge variant="success">Active</Badge>
  </div>

  <div className="mt-4 flex items-center gap-4">
    <Metric label="Views" value="1,234" />
    <Metric label="Conversions" value="56" />
  </div>
</Card>
```

### Action Card (Clickable)
```tsx
<Card clickable onClick={() => navigate("/details")}>
  <div className="flex items-center gap-4">
    <div className="flex h-12 w-12 items-center justify-center rounded-lg bg-primary-100 dark:bg-primary-900/20">
      <Icon className="h-6 w-6 text-primary-600 dark:text-primary-400" />
    </div>

    <div className="flex-1">
      <h3 className="font-medium text-neutral-900 dark:text-neutral-100">
        Feature Name
      </h3>
      <p className="text-sm text-neutral-500 dark:text-neutral-400">
        Click to learn more
      </p>
    </div>

    <ChevronRight className="h-5 w-5 text-neutral-400" />
  </div>
</Card>
```

## Innovation Opportunities

1. **Gradient Border on Hover**: Subtle animated gradient
2. **Loading Skeleton**: Pulse animation while loading content
3. **Expandable**: Smooth height animation to reveal more
4. **Drag to Reorder**: With visual feedback
5. **Swipe Actions**: Mobile swipe to reveal actions

## Anti-Patterns to Avoid

- ❌ Uniform card sizes in a dynamic grid (be intentional)
- ❌ Clickable cards with no hover feedback
- ❌ Shadow that's too strong (keep subtle)
- ❌ No focus state on interactive cards
- ❌ Rounded corners that don't match design system
