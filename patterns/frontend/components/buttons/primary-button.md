# Primary Button Pattern

## Reference Sources
- **Linear**: Spring animation, gradient border on focus
- **Remotion**: Clear hierarchy, brand color prominence
- **Stripe**: Perfect padding, subtle shadow

## States

### Default
- Background: `primary-600`
- Text: `white`
- Shadow: `sm`
- Border-radius: design system `md`

### Hover
- Background: `primary-700`
- Transform: `scale(1.02)`
- Shadow: `md`
- Transition: 150ms spring

### Focus
- Ring: 2px `primary-500`, 2px offset
- Background: `primary-600` (same as default)

### Active/Pressed
- Transform: `scale(0.98)`
- Shadow: `sm` (reduced)
- Transition: 50ms

### Disabled
- Background: `neutral-200`
- Text: `neutral-400`
- Cursor: `not-allowed`
- No hover effects

### Loading
- Content replaced with spinner
- Maintains button width
- Spinner color: `currentColor`

## Code Example (React + Tailwind)

```tsx
import { cn } from "@/lib/utils";
import { Loader2 } from "lucide-react";

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  isLoading?: boolean;
  variant?: "primary" | "secondary" | "outline";
}

const Button = ({ children, isLoading, variant = "primary", className, ...props }: ButtonProps) => (
  <button
    className={cn(
      // Base
      "inline-flex items-center justify-center",
      "px-4 py-2 text-sm font-medium",
      "rounded-md shadow-sm",

      // Primary variant
      variant === "primary" && [
        "bg-primary-600 text-white",
        "hover:bg-primary-700 hover:scale-[1.02] hover:shadow-md",
      ],

      // Focus
      "focus:outline-none focus-visible:ring-2",
      "focus-visible:ring-primary-500 focus-visible:ring-offset-2",

      // Active
      "active:scale-[0.98] active:shadow-sm",

      // Transition
      "transition-all duration-150",
      "motion-safe:transition-transform",

      // Disabled
      "disabled:bg-neutral-200 disabled:text-neutral-400",
      "disabled:cursor-not-allowed disabled:hover:scale-100",

      className
    )}
    disabled={isLoading || props.disabled}
    {...props}
  >
    {isLoading ? (
      <Loader2 className="h-4 w-4 animate-spin" />
    ) : (
      children
    )}
  </button>
);

export default Button;
```

## Framer Motion Variant

```tsx
import { motion } from "framer-motion";

const buttonVariants = {
  initial: { scale: 1 },
  hover: { scale: 1.02 },
  tap: { scale: 0.98 },
};

const springTransition = {
  type: "spring",
  stiffness: 400,
  damping: 17
};

const MotionButton = ({ children, ...props }) => (
  <motion.button
    variants={buttonVariants}
    initial="initial"
    whileHover="hover"
    whileTap="tap"
    transition={springTransition}
    className="px-4 py-2 bg-primary-600 text-white rounded-md"
    {...props}
  >
    {children}
  </motion.button>
);
```

## Innovation Opportunities

1. **Haptic Feedback**: Add subtle vibration on mobile
2. **Success Animation**: Brief scale pulse on successful action
3. **Loading Progress**: Show percentage in loading state
4. **Keyboard Shortcuts**: Display shortcut hint on focus
5. **Sound Design**: Subtle audio feedback (optional)

## Anti-Patterns to Avoid

- ❌ Hover effect that only changes color
- ❌ No focus state
- ❌ Linear transition timing
- ❌ No loading state
- ❌ Disabled state that looks clickable
