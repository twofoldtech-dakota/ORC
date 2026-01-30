# Remotion API Reference

Complete reference for Remotion hooks, components, and utilities.

---

## Hooks

### useCurrentFrame

Returns the current frame number (0-indexed).

```tsx
import { useCurrentFrame } from 'remotion';

const MyComponent: React.FC = () => {
  const frame = useCurrentFrame();
  // frame = 0 at start, increments each frame
  return <div>Frame: {frame}</div>;
};
```

### useVideoConfig

Returns the video configuration object.

```tsx
import { useVideoConfig } from 'remotion';

const MyComponent: React.FC = () => {
  const { fps, durationInFrames, width, height, id, defaultProps } = useVideoConfig();

  const durationSeconds = durationInFrames / fps;
  return <div>Duration: {durationSeconds}s</div>;
};
```

**Returns:**
| Property | Type | Description |
|----------|------|-------------|
| `fps` | `number` | Frames per second |
| `durationInFrames` | `number` | Total frames |
| `width` | `number` | Composition width in pixels |
| `height` | `number` | Composition height in pixels |
| `id` | `string` | Composition ID |
| `defaultProps` | `object` | Default props passed to composition |

---

## Animation Functions

### interpolate

Maps an input value from one range to another.

```tsx
import { interpolate } from 'remotion';

const frame = useCurrentFrame();

// Basic: frame 0-30 maps to opacity 0-1
const opacity = interpolate(frame, [0, 30], [0, 1]);

// With clamping (recommended)
const opacity = interpolate(frame, [0, 30], [0, 1], {
  extrapolateLeft: 'clamp',
  extrapolateRight: 'clamp',
});

// With easing
import { Easing } from 'remotion';
const opacity = interpolate(frame, [0, 30], [0, 1], {
  easing: Easing.bezier(0.25, 0.1, 0.25, 1),
  extrapolateRight: 'clamp',
});
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `input` | `number` | Current value (usually frame) |
| `inputRange` | `number[]` | Input range [start, end] |
| `outputRange` | `number[]` | Output range [start, end] |
| `options?` | `object` | Optional configuration |

**Options:**
| Option | Values | Description |
|--------|--------|-------------|
| `extrapolateLeft` | `'extend'` \| `'clamp'` \| `'identity'` | Behavior before input range |
| `extrapolateRight` | `'extend'` \| `'clamp'` \| `'identity'` | Behavior after input range |
| `easing` | `(t: number) => number` | Easing function |

### interpolateColors

Interpolates between colors.

```tsx
import { interpolateColors } from 'remotion';

const frame = useCurrentFrame();
const color = interpolateColors(
  frame,
  [0, 30, 60],
  ['#ff0000', '#00ff00', '#0000ff']
);

return <div style={{ backgroundColor: color }} />;
```

**Supports:** Hex, RGB, RGBA, HSL, HSLA color formats.

### spring

Creates physics-based spring animation.

```tsx
import { spring, useCurrentFrame, useVideoConfig } from 'remotion';

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

const scale = spring({
  frame,
  fps,
  config: {
    damping: 10,
    stiffness: 100,
    mass: 1,
  },
});

return <div style={{ transform: `scale(${scale})` }} />;
```

**Parameters:**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `frame` | `number` | required | Current frame |
| `fps` | `number` | required | Frames per second |
| `config` | `object` | `{}` | Spring configuration |
| `from` | `number` | `0` | Starting value |
| `to` | `number` | `1` | Ending value |
| `durationInFrames` | `number` | auto | Limit duration |
| `durationRestThreshold` | `number` | `0.001` | When to consider settled |
| `delay` | `number` | `0` | Delay in frames |

**Config options:**
| Option | Default | Description |
|--------|---------|-------------|
| `damping` | `10` | Friction (higher = less bounce) |
| `stiffness` | `100` | Spring tension (higher = faster) |
| `mass` | `1` | Object weight (higher = slower) |
| `overshootClamping` | `false` | Clamp overshoot |

### measureSpring

Calculate how many frames a spring animation takes.

```tsx
import { measureSpring } from 'remotion';

const duration = measureSpring({
  fps: 30,
  config: { damping: 10, stiffness: 100 },
});
// Returns number of frames until spring settles
```

### Easing

Built-in easing functions for `interpolate`.

```tsx
import { Easing, interpolate } from 'remotion';

// Predefined easings
Easing.linear
Easing.ease
Easing.quad
Easing.cubic
Easing.sin
Easing.circle
Easing.exp
Easing.elastic(1)
Easing.back(1.5)
Easing.bounce

// Bezier curve
Easing.bezier(0.25, 0.1, 0.25, 1)

// Modifiers
Easing.in(Easing.quad)
Easing.out(Easing.quad)
Easing.inOut(Easing.quad)
```

---

## Components

### Composition

Registers a video composition.

```tsx
import { Composition } from 'remotion';

<Composition
  id="MyVideo"                    // Unique identifier
  component={MyVideoComponent}    // React component
  durationInFrames={300}          // Total frames
  fps={30}                        // Frames per second
  width={1920}                    // Width in pixels
  height={1080}                   // Height in pixels
  defaultProps={{                 // Optional default props
    title: 'Hello',
  }}
/>
```

### Still

Registers a still image composition.

```tsx
import { Still } from 'remotion';

<Still
  id="Thumbnail"
  component={ThumbnailComponent}
  width={1920}
  height={1080}
  defaultProps={{ title: 'My Thumbnail' }}
/>
```

### Sequence

Time-shifts children to start at a specific frame.

```tsx
import { Sequence } from 'remotion';

<Sequence
  from={30}              // Start frame (required)
  durationInFrames={60}  // Optional duration
  name="Intro"           // Optional label for Studio
  layout="none"          // Optional: 'none' | 'absolute-fill'
>
  <ChildComponent />
</Sequence>
```

**Behavior:**
- Children see frame 0 when parent is at `from`
- `useCurrentFrame()` inside returns frames relative to Sequence start
- Sequence is invisible before `from` and after `from + durationInFrames`

### Series

Sequential arrangement of clips.

```tsx
import { Series } from 'remotion';

<Series>
  <Series.Sequence durationInFrames={60}>
    <Intro />
  </Series.Sequence>
  <Series.Sequence durationInFrames={120}>
    <MainContent />
  </Series.Sequence>
  <Series.Sequence durationInFrames={60}>
    <Outro />
  </Series.Sequence>
</Series>
```

**With offset (gaps/overlaps):**
```tsx
<Series>
  <Series.Sequence durationInFrames={60}>
    <ClipOne />
  </Series.Sequence>
  <Series.Sequence durationInFrames={60} offset={-10}>
    {/* Starts 10 frames before ClipOne ends (overlap) */}
    <ClipTwo />
  </Series.Sequence>
  <Series.Sequence durationInFrames={60} offset={15}>
    {/* Starts 15 frames after ClipTwo ends (gap) */}
    <ClipThree />
  </Series.Sequence>
</Series>
```

### Loop

Repeats content.

```tsx
import { Loop } from 'remotion';

<Loop durationInFrames={30} times={4}>
  {/* Repeats 4 times, each iteration is 30 frames */}
  <BouncingBall />
</Loop>

<Loop durationInFrames={30}>
  {/* Repeats infinitely for composition duration */}
  <BouncingBall />
</Loop>
```

### AbsoluteFill

Full-frame absolutely positioned container.

```tsx
import { AbsoluteFill } from 'remotion';

<AbsoluteFill
  style={{
    backgroundColor: '#000',
    justifyContent: 'center',
    alignItems: 'center',
  }}
>
  <h1>Centered Content</h1>
</AbsoluteFill>
```

Equivalent to:
```tsx
<div style={{
  position: 'absolute',
  top: 0,
  left: 0,
  right: 0,
  bottom: 0,
}}>
```

### Img

Optimized image component.

```tsx
import { Img, staticFile } from 'remotion';

<Img
  src={staticFile('logo.png')}
  style={{ width: 200 }}
  onError={(e) => console.error('Image failed to load')}
/>
```

### Video

Embed video with synchronization.

```tsx
import { Video, staticFile } from 'remotion';

<Video
  src={staticFile('clip.mp4')}
  volume={0.5}              // 0 to 1
  playbackRate={1}          // Speed multiplier
  startFrom={30}            // Trim start (frames)
  endAt={150}               // Trim end (frames)
  muted={false}
  loop={false}
/>
```

### OffthreadVideo

Memory-efficient video for heavy compositions.

```tsx
import { OffthreadVideo, staticFile } from 'remotion';

<OffthreadVideo
  src={staticFile('heavy-video.mp4')}
  volume={1}
  playbackRate={1}
/>
```

**Use when:**
- Embedding multiple videos
- Working with 4K+ footage
- Experiencing memory issues

### Audio

Embed audio with synchronization.

```tsx
import { Audio, staticFile } from 'remotion';

<Audio
  src={staticFile('music.mp3')}
  volume={0.8}
  startFrom={0}
  endAt={300}
/>
```

**Dynamic volume:**
```tsx
const frame = useCurrentFrame();
const volume = interpolate(frame, [0, 30], [0, 1], {
  extrapolateRight: 'clamp',
});

<Audio src={staticFile('music.mp3')} volume={volume} />
```

### IFrame

Embed iframe content.

```tsx
import { IFrame } from 'remotion';

<IFrame
  src="https://example.com"
  style={{ width: '100%', height: '100%' }}
/>
```

---

## Utilities

### staticFile

Reference files in the `public/` directory.

```tsx
import { staticFile } from 'remotion';

const logoSrc = staticFile('images/logo.png');
// Resolves to correct path during development and render
```

**File structure:**
```
project/
├── public/
│   ├── images/
│   │   └── logo.png    → staticFile('images/logo.png')
│   └── audio/
│       └── music.mp3   → staticFile('audio/music.mp3')
└── src/
```

### delayRender / continueRender

Handle async operations during render.

```tsx
import { delayRender, continueRender } from 'remotion';
import { useState, useEffect } from 'react';

const AsyncComponent: React.FC = () => {
  const [data, setData] = useState(null);
  const [handle] = useState(() => delayRender('Loading API data'));

  useEffect(() => {
    fetchData()
      .then((result) => {
        setData(result);
        continueRender(handle);
      })
      .catch((err) => {
        // Important: still call continueRender to unblock
        console.error(err);
        continueRender(handle);
      });
  }, [handle]);

  if (!data) return null;
  return <div>{data.content}</div>;
};
```

### getInputProps

Get props passed via CLI.

```tsx
import { getInputProps } from 'remotion';

// CLI: npx remotion render ... --props='{"title":"Hello"}'
const props = getInputProps<{ title: string }>();
console.log(props.title); // "Hello"
```

### random

Deterministic random number (same seed = same result).

```tsx
import { random } from 'remotion';

// Returns same value for same seed across renders
const value = random('my-seed'); // 0 to 1
const value2 = random(42);       // Also works with numbers

// Use for consistent randomness
const particles = Array.from({ length: 100 }, (_, i) => ({
  x: random(`particle-${i}-x`) * 1920,
  y: random(`particle-${i}-y`) * 1080,
}));
```

### getRemotionEnvironment

Check current execution environment.

```tsx
import { getRemotionEnvironment } from 'remotion';

const env = getRemotionEnvironment();

if (env.isStudio) {
  // Running in Remotion Studio
}
if (env.isRendering) {
  // Being rendered via CLI
}
if (env.isPlayer) {
  // Running in Remotion Player
}
```

---

## Configuration (remotion.config.ts)

```ts
import { Config } from '@remotion/cli/config';

// Rendering
Config.setOutputLocation('out/video.mp4');
Config.setConcurrency(8);
Config.setCodec('h264');
Config.setCrf(18);

// Video
Config.setScale(1);
Config.setMuted(false);

// Image sequences
Config.setImageFormat('png');
Config.setImageSequence(false);

// Browser
Config.setChromiumOpenGlRenderer('angle');
```

---

## TypeScript Types

```tsx
import type {
  VideoConfig,
  SpringConfig,
  InterpolateOptions,
} from 'remotion';

// Component props with video config
interface MyComponentProps {
  title: string;
}

const MyComponent: React.FC<MyComponentProps> = ({ title }) => {
  const config: VideoConfig = useVideoConfig();
  return <div>{title}</div>;
};

// Spring config
const springConfig: SpringConfig = {
  damping: 10,
  stiffness: 100,
  mass: 1,
};
```
