# Text Input Pattern

## Reference Sources
- **Stripe**: Clear labels, helpful validation, smooth transitions
- **Linear**: Floating labels, subtle focus animations
- **Vercel**: Minimal design with clear states

## States

### Default (Empty)
- Border: 1px `neutral-300` / `neutral-700` (dark)
- Background: `white` / `neutral-900` (dark)
- Label: `neutral-700` / `neutral-300` (dark)
- Placeholder: `neutral-400`

### Focus
- Border: 2px `primary-500`
- Ring: subtle glow matching border color
- Label: `primary-600` (if floating)
- Transition: 150ms ease-out

### Filled (Valid)
- Border: `neutral-300` / `neutral-700` (dark)
- Label: stays elevated (if floating)

### Error
- Border: 2px `error-500`
- Label: `error-600`
- Helper text: `error-600` with icon
- Optional: shake animation on validation

### Success (if shown)
- Border: `success-500`
- Icon: checkmark in `success-600`
- Optional: micro-celebration animation

### Disabled
- Background: `neutral-100` / `neutral-800` (dark)
- Text: `neutral-400`
- Cursor: `not-allowed`

## Code Example (React + Tailwind)

```tsx
import { cn } from "@/lib/utils";
import { AlertCircle, CheckCircle2 } from "lucide-react";
import { useState } from "react";

interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
  success?: boolean;
  helperText?: string;
}

const Input = ({
  label,
  error,
  success,
  helperText,
  className,
  ...props
}: InputProps) => {
  const [isFocused, setIsFocused] = useState(false);
  const hasValue = props.value || props.defaultValue;

  return (
    <div className="w-full">
      <div className="relative">
        {/* Floating Label */}
        <label
          htmlFor={props.id}
          className={cn(
            "absolute left-3 transition-all duration-150",
            "pointer-events-none",
            isFocused || hasValue
              ? "top-0 -translate-y-1/2 text-xs bg-white dark:bg-neutral-900 px-1"
              : "top-1/2 -translate-y-1/2 text-sm",
            error
              ? "text-error-600 dark:text-error-400"
              : isFocused
              ? "text-primary-600 dark:text-primary-400"
              : "text-neutral-700 dark:text-neutral-300"
          )}
        >
          {label}
        </label>

        {/* Input */}
        <input
          {...props}
          onFocus={(e) => {
            setIsFocused(true);
            props.onFocus?.(e);
          }}
          onBlur={(e) => {
            setIsFocused(false);
            props.onBlur?.(e);
          }}
          className={cn(
            "block w-full px-3 py-2",
            "border rounded-md",
            "transition-all duration-150",
            "placeholder:text-neutral-400",

            // States
            error
              ? "border-error-500 focus:border-error-500 focus:ring-2 focus:ring-error-500/20"
              : success
              ? "border-success-500 focus:border-success-500 focus:ring-2 focus:ring-success-500/20"
              : "border-neutral-300 dark:border-neutral-700 focus:border-primary-500 focus:ring-2 focus:ring-primary-500/20",

            // Disabled
            "disabled:bg-neutral-100 dark:disabled:bg-neutral-800",
            "disabled:cursor-not-allowed disabled:text-neutral-400",

            className
          )}
        />

        {/* Success Icon */}
        {success && !error && (
          <div className="absolute right-3 top-1/2 -translate-y-1/2">
            <CheckCircle2 className="h-5 w-5 text-success-600" />
          </div>
        )}
      </div>

      {/* Helper Text / Error Message */}
      {(helperText || error) && (
        <div className={cn(
          "mt-1.5 flex items-start gap-1 text-sm",
          error ? "text-error-600 dark:text-error-400" : "text-neutral-500 dark:text-neutral-400"
        )}>
          {error && <AlertCircle className="h-4 w-4 mt-0.5 flex-shrink-0" />}
          <p>{error || helperText}</p>
        </div>
      )}
    </div>
  );
};

export default Input;
```

## Advanced Pattern: Floating Label with Animation

```tsx
import { motion } from "framer-motion";

const FloatingInput = ({ label, ...props }) => {
  const [isFocused, setIsFocused] = useState(false);
  const hasValue = !!props.value;

  return (
    <div className="relative">
      <motion.label
        animate={{
          y: isFocused || hasValue ? -20 : 0,
          scale: isFocused || hasValue ? 0.85 : 1,
          color: isFocused ? "rgb(59, 130, 246)" : "rgb(115, 115, 115)"
        }}
        transition={{ type: "spring", stiffness: 300, damping: 20 }}
        className="absolute left-3 top-1/2 -translate-y-1/2 origin-left pointer-events-none"
      >
        {label}
      </motion.label>

      <input
        {...props}
        onFocus={(e) => {
          setIsFocused(true);
          props.onFocus?.(e);
        }}
        onBlur={(e) => {
          setIsFocused(false);
          props.onBlur?.(e);
        }}
        className="..."
      />
    </div>
  );
};
```

## Validation Patterns

### Inline Validation (Real-time)
```tsx
const [email, setEmail] = useState("");
const [touched, setTouched] = useState(false);

const isValid = /\S+@\S+\.\S+/.test(email);
const showError = touched && email && !isValid;
const showSuccess = touched && isValid;

return (
  <Input
    label="Email"
    value={email}
    onChange={(e) => setEmail(e.target.value)}
    onBlur={() => setTouched(true)}
    error={showError ? "Please enter a valid email address" : undefined}
    success={showSuccess}
  />
);
```

## Innovation Opportunities

1. **Password Strength Meter**: Visual indicator with color progression
2. **Auto-complete with Preview**: Show suggestions with hover preview
3. **Voice Input**: Speech-to-text with animation
4. **Smart Validation**: Suggest corrections (did you mean .com?)
5. **Format on Blur**: Auto-format phone numbers, dates

## Anti-Patterns to Avoid

- ❌ Validation only on submit (validate as user types)
- ❌ Error message without explanation
- ❌ No focus state
- ❌ Placeholder as label (accessibility issue)
- ❌ Red border without message explaining the error
