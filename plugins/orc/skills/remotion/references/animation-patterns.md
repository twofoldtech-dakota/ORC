# Remotion Animation Patterns

Reusable animation recipes with complete code examples.

---

## Basic Animations

### Fade In

```tsx
import { useCurrentFrame, interpolate, AbsoluteFill } from 'remotion';

export const FadeIn: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return <div style={{ opacity }}>{children}</div>;
};
```

### Fade Out

```tsx
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const FadeOut: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [durationInFrames - 30, durationInFrames],
    [1, 0],
    { extrapolateLeft: 'clamp' }
  );

  return <div style={{ opacity }}>{children}</div>;
};
```

### Fade In and Out

```tsx
export const FadeInOut: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [0, 20, durationInFrames - 20, durationInFrames],
    [0, 1, 1, 0],
    { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
  );

  return <div style={{ opacity }}>{children}</div>;
};
```

---

## Slide Animations

### Slide From Left

```tsx
export const SlideFromLeft: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const translateX = interpolate(frame, [0, 30], [-100, 0], {
    extrapolateRight: 'clamp',
  });

  return (
    <div style={{ transform: `translateX(${translateX}%)` }}>
      {children}
    </div>
  );
};
```

### Slide From Right

```tsx
export const SlideFromRight: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const translateX = interpolate(frame, [0, 30], [100, 0], {
    extrapolateRight: 'clamp',
  });

  return (
    <div style={{ transform: `translateX(${translateX}%)` }}>
      {children}
    </div>
  );
};
```

### Slide From Bottom

```tsx
export const SlideFromBottom: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const translateY = interpolate(frame, [0, 30], [100, 0], {
    extrapolateRight: 'clamp',
  });

  return (
    <div style={{ transform: `translateY(${translateY}%)` }}>
      {children}
    </div>
  );
};
```

### Slide From Top

```tsx
export const SlideFromTop: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const translateY = interpolate(frame, [0, 30], [-100, 0], {
    extrapolateRight: 'clamp',
  });

  return (
    <div style={{ transform: `translateY(${translateY}%)` }}>
      {children}
    </div>
  );
};
```

---

## Scale Animations

### Scale In (Linear)

```tsx
export const ScaleIn: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const scale = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <div style={{ transform: `scale(${scale})` }}>
      {children}
    </div>
  );
};
```

### Scale In (Spring)

```tsx
import { spring, useCurrentFrame, useVideoConfig } from 'remotion';

export const SpringScaleIn: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const scale = spring({
    frame,
    fps,
    config: {
      damping: 12,
      stiffness: 200,
    },
  });

  return (
    <div style={{ transform: `scale(${scale})` }}>
      {children}
    </div>
  );
};
```

### Zoom In (Ken Burns Effect)

```tsx
export const KenBurns: React.FC<{ src: string }> = ({ src }) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();

  const scale = interpolate(frame, [0, durationInFrames], [1, 1.3]);
  const translateX = interpolate(frame, [0, durationInFrames], [0, -5]);
  const translateY = interpolate(frame, [0, durationInFrames], [0, -3]);

  return (
    <AbsoluteFill>
      <Img
        src={src}
        style={{
          width: '100%',
          height: '100%',
          objectFit: 'cover',
          transform: `scale(${scale}) translate(${translateX}%, ${translateY}%)`,
        }}
      />
    </AbsoluteFill>
  );
};
```

---

## Spring Animations

### Bounce In

```tsx
export const BounceIn: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const scale = spring({
    frame,
    fps,
    config: {
      damping: 8,      // Less damping = more bounce
      stiffness: 200,
    },
  });

  return (
    <div style={{ transform: `scale(${scale})` }}>
      {children}
    </div>
  );
};
```

### Elastic Pop

```tsx
export const ElasticPop: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const scale = spring({
    frame,
    fps,
    config: {
      damping: 6,
      stiffness: 150,
      mass: 0.5,
    },
  });

  return (
    <div style={{ transform: `scale(${scale})` }}>
      {children}
    </div>
  );
};
```

### Slide with Spring

```tsx
export const SpringSlide: React.FC<{
  children: React.ReactNode;
  from?: 'left' | 'right' | 'top' | 'bottom';
}> = ({ children, from = 'left' }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const progress = spring({
    frame,
    fps,
    config: { damping: 15, stiffness: 100 },
  });

  const transforms = {
    left: `translateX(${interpolate(progress, [0, 1], [-100, 0])}%)`,
    right: `translateX(${interpolate(progress, [0, 1], [100, 0])}%)`,
    top: `translateY(${interpolate(progress, [0, 1], [-100, 0])}%)`,
    bottom: `translateY(${interpolate(progress, [0, 1], [100, 0])}%)`,
  };

  return (
    <div style={{ transform: transforms[from] }}>
      {children}
    </div>
  );
};
```

---

## Text Animations

### Typewriter Effect

```tsx
export const Typewriter: React.FC<{ text: string; speed?: number }> = ({
  text,
  speed = 2 // frames per character
}) => {
  const frame = useCurrentFrame();
  const charsToShow = Math.floor(frame / speed);

  return (
    <span style={{ fontFamily: 'monospace' }}>
      {text.slice(0, Math.min(charsToShow, text.length))}
      <span style={{ opacity: frame % 30 < 15 ? 1 : 0 }}>|</span>
    </span>
  );
};
```

### Word by Word Reveal

```tsx
export const WordReveal: React.FC<{ text: string; delay?: number }> = ({
  text,
  delay = 10 // frames between words
}) => {
  const frame = useCurrentFrame();
  const words = text.split(' ');

  return (
    <span>
      {words.map((word, i) => {
        const wordFrame = frame - i * delay;
        const opacity = interpolate(wordFrame, [0, 10], [0, 1], {
          extrapolateLeft: 'clamp',
          extrapolateRight: 'clamp',
        });

        return (
          <span key={i} style={{ opacity }}>
            {word}{' '}
          </span>
        );
      })}
    </span>
  );
};
```

### Character Stagger

```tsx
export const CharacterStagger: React.FC<{ text: string }> = ({ text }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <span>
      {text.split('').map((char, i) => {
        const delay = i * 2;
        const scale = spring({
          frame: frame - delay,
          fps,
          config: { damping: 12, stiffness: 200 },
        });

        return (
          <span
            key={i}
            style={{
              display: 'inline-block',
              transform: `scale(${scale})`,
              opacity: scale,
            }}
          >
            {char === ' ' ? '\u00A0' : char}
          </span>
        );
      })}
    </span>
  );
};
```

---

## Staggered Animations

### Staggered List

```tsx
import { Sequence } from 'remotion';

export const StaggeredList: React.FC<{
  items: React.ReactNode[];
  staggerDelay?: number;
}> = ({ items, staggerDelay = 10 }) => {
  return (
    <div>
      {items.map((item, i) => (
        <Sequence key={i} from={i * staggerDelay}>
          <FadeIn>
            <SlideFromLeft>{item}</SlideFromLeft>
          </FadeIn>
        </Sequence>
      ))}
    </div>
  );
};
```

### Grid Stagger

```tsx
export const StaggeredGrid: React.FC<{
  items: React.ReactNode[];
  columns: number;
  staggerDelay?: number;
}> = ({ items, columns, staggerDelay = 5 }) => {
  return (
    <div style={{
      display: 'grid',
      gridTemplateColumns: `repeat(${columns}, 1fr)`,
      gap: 20,
    }}>
      {items.map((item, i) => {
        const row = Math.floor(i / columns);
        const col = i % columns;
        const delay = (row + col) * staggerDelay;

        return (
          <Sequence key={i} from={delay}>
            <SpringScaleIn>{item}</SpringScaleIn>
          </Sequence>
        );
      })}
    </div>
  );
};
```

---

## Progress & Data Visualization

### Animated Progress Bar

```tsx
export const ProgressBar: React.FC<{
  progress: number; // 0 to 1
  color?: string;
}> = ({ progress, color = '#3b82f6' }) => {
  const frame = useCurrentFrame();

  const animatedProgress = interpolate(
    frame,
    [0, 60],
    [0, progress],
    { extrapolateRight: 'clamp' }
  );

  return (
    <div style={{
      width: '100%',
      height: 20,
      backgroundColor: '#1f2937',
      borderRadius: 10,
      overflow: 'hidden',
    }}>
      <div style={{
        width: `${animatedProgress * 100}%`,
        height: '100%',
        backgroundColor: color,
        borderRadius: 10,
      }} />
    </div>
  );
};
```

### Animated Counter

```tsx
export const Counter: React.FC<{
  from?: number;
  to: number;
  duration?: number; // in frames
}> = ({ from = 0, to, duration = 60 }) => {
  const frame = useCurrentFrame();

  const value = interpolate(
    frame,
    [0, duration],
    [from, to],
    { extrapolateRight: 'clamp' }
  );

  return <span>{Math.round(value)}</span>;
};
```

### Circular Progress

```tsx
export const CircularProgress: React.FC<{
  progress: number;
  size?: number;
  strokeWidth?: number;
  color?: string;
}> = ({ progress, size = 100, strokeWidth = 8, color = '#3b82f6' }) => {
  const frame = useCurrentFrame();

  const animatedProgress = interpolate(
    frame,
    [0, 60],
    [0, progress],
    { extrapolateRight: 'clamp' }
  );

  const radius = (size - strokeWidth) / 2;
  const circumference = 2 * Math.PI * radius;
  const offset = circumference * (1 - animatedProgress);

  return (
    <svg width={size} height={size}>
      <circle
        cx={size / 2}
        cy={size / 2}
        r={radius}
        fill="none"
        stroke="#1f2937"
        strokeWidth={strokeWidth}
      />
      <circle
        cx={size / 2}
        cy={size / 2}
        r={radius}
        fill="none"
        stroke={color}
        strokeWidth={strokeWidth}
        strokeDasharray={circumference}
        strokeDashoffset={offset}
        strokeLinecap="round"
        transform={`rotate(-90 ${size / 2} ${size / 2})`}
      />
    </svg>
  );
};
```

---

## Transitions

### Cross-Fade Transition

```tsx
export const CrossFade: React.FC<{
  children: [React.ReactNode, React.ReactNode];
  transitionFrame: number;
  transitionDuration?: number;
}> = ({ children, transitionFrame, transitionDuration = 30 }) => {
  const frame = useCurrentFrame();

  const opacity1 = interpolate(
    frame,
    [transitionFrame, transitionFrame + transitionDuration],
    [1, 0],
    { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
  );

  const opacity2 = interpolate(
    frame,
    [transitionFrame, transitionFrame + transitionDuration],
    [0, 1],
    { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill>
      <AbsoluteFill style={{ opacity: opacity1 }}>{children[0]}</AbsoluteFill>
      <AbsoluteFill style={{ opacity: opacity2 }}>{children[1]}</AbsoluteFill>
    </AbsoluteFill>
  );
};
```

### Wipe Transition

```tsx
export const WipeTransition: React.FC<{
  children: [React.ReactNode, React.ReactNode];
  transitionFrame: number;
  transitionDuration?: number;
  direction?: 'left' | 'right' | 'up' | 'down';
}> = ({
  children,
  transitionFrame,
  transitionDuration = 30,
  direction = 'right',
}) => {
  const frame = useCurrentFrame();

  const progress = interpolate(
    frame,
    [transitionFrame, transitionFrame + transitionDuration],
    [0, 100],
    { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
  );

  const clipPaths = {
    right: `inset(0 ${100 - progress}% 0 0)`,
    left: `inset(0 0 0 ${100 - progress}%)`,
    down: `inset(0 0 ${100 - progress}% 0)`,
    up: `inset(${100 - progress}% 0 0 0)`,
  };

  return (
    <AbsoluteFill>
      <AbsoluteFill>{children[0]}</AbsoluteFill>
      <AbsoluteFill style={{ clipPath: clipPaths[direction] }}>
        {children[1]}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

---

## Rotation & Movement

### Continuous Rotation

```tsx
export const Rotate: React.FC<{
  children: React.ReactNode;
  duration?: number; // frames per full rotation
}> = ({ children, duration = 60 }) => {
  const frame = useCurrentFrame();
  const rotation = (frame / duration) * 360;

  return (
    <div style={{ transform: `rotate(${rotation}deg)` }}>
      {children}
    </div>
  );
};
```

### Orbit Animation

```tsx
export const Orbit: React.FC<{
  children: React.ReactNode;
  radius?: number;
  duration?: number;
}> = ({ children, radius = 100, duration = 90 }) => {
  const frame = useCurrentFrame();
  const angle = (frame / duration) * 2 * Math.PI;

  const x = Math.cos(angle) * radius;
  const y = Math.sin(angle) * radius;

  return (
    <div style={{ transform: `translate(${x}px, ${y}px)` }}>
      {children}
    </div>
  );
};
```

### Parallax Effect

```tsx
export const ParallaxLayer: React.FC<{
  children: React.ReactNode;
  speed: number; // 0.5 = slower, 2 = faster
}> = ({ children, speed }) => {
  const frame = useCurrentFrame();
  const { durationInFrames, height } = useVideoConfig();

  const scrollProgress = frame / durationInFrames;
  const translateY = scrollProgress * height * (1 - speed);

  return (
    <div style={{ transform: `translateY(${-translateY}px)` }}>
      {children}
    </div>
  );
};
```

---

## Color Animations

### Color Transition

```tsx
import { interpolateColors } from 'remotion';

export const ColorTransition: React.FC<{
  children: React.ReactNode;
  colors: string[];
  property?: 'backgroundColor' | 'color';
}> = ({ children, colors, property = 'backgroundColor' }) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();

  const inputRange = colors.map((_, i) =>
    (i / (colors.length - 1)) * durationInFrames
  );

  const color = interpolateColors(frame, inputRange, colors);

  return (
    <div style={{ [property]: color }}>
      {children}
    </div>
  );
};
```

### Gradient Animation

```tsx
export const AnimatedGradient: React.FC = () => {
  const frame = useCurrentFrame();
  const rotation = interpolate(frame, [0, 120], [0, 360]);

  return (
    <AbsoluteFill
      style={{
        background: `linear-gradient(${rotation}deg, #667eea, #764ba2, #6B8DD6)`,
      }}
    />
  );
};
```

---

## Composition Helpers

### Combine Animations

```tsx
// Utility to combine multiple animation wrappers
export const withAnimations = (
  component: React.ReactNode,
  ...wrappers: React.FC<{ children: React.ReactNode }>[]
) => {
  return wrappers.reduce(
    (acc, Wrapper) => <Wrapper>{acc}</Wrapper>,
    component
  );
};

// Usage
const AnimatedElement = withAnimations(
  <div>Content</div>,
  FadeIn,
  SlideFromLeft
);
```

### Delayed Animation

```tsx
export const Delayed: React.FC<{
  children: React.ReactNode;
  delay: number;
}> = ({ children, delay }) => {
  const frame = useCurrentFrame();

  if (frame < delay) return null;

  return (
    <Sequence from={0}>
      {children}
    </Sequence>
  );
};
```
