---
name: remotion
description: "Remotion video creation - programmatically create videos using React. Use for: (1) animated videos, motion graphics, or video content from code, (2) data-driven or templated video generation systems, (3) social media videos, product demos, explainer animations, marketing content, (4) .mp4, .webm, .gif, or image sequence outputs, (5) React-based video compositions with animations, transitions, and effects. Covers Composition, Sequence, useCurrentFrame, useVideoConfig, interpolate, spring animations, and npx remotion render."
version: 1.0.0
triggers:
  - "/remotion"
---

# Remotion Video Creation

Create videos programmatically using React components. Each frame is a React render; animations derive from the current frame number.

## Quick Reference

| Task | Resource |
|------|----------|
| Project setup | [Setup Commands](#setup-commands) |
| Create composition | [Core Concepts](#core-concepts) |
| Add animations | [Animation Patterns](#animation-patterns) / [references/animation-patterns.md](references/animation-patterns.md) |
| API lookup | [Essential APIs](#essential-apis) / [references/api-reference.md](references/api-reference.md) |
| Render video | [Rendering Commands](#rendering-commands) |
| Debug issues | [references/troubleshooting.md](references/troubleshooting.md) |

---

## Core Concepts

### Compositions
The video container defining dimensions and duration:
```tsx
<Composition
  id="MyVideo"
  component={MyComponent}
  durationInFrames={300}  // 10 seconds at 30fps
  fps={30}
  width={1920}
  height={1080}
/>
```

### Frames
Time unit in Remotion. Frame 0 = start, `durationInFrames - 1` = end.
- Convert seconds to frames: `seconds * fps`
- Convert frames to seconds: `frame / fps`

### Sequences
Time-shift children to start at a specific frame:
```tsx
<Sequence from={30} durationInFrames={60}>
  <SceneTwo />  {/* Appears at frame 30, lasts 60 frames */}
</Sequence>
```

### Component Model
- Components must be **pure**: same frame = same output
- Use `useCurrentFrame()` to get current frame
- Derive all animated values from frame number
- No `useState` for animation state

---

## Setup Commands

### New Project
```bash
npx create-video@latest my-video
cd my-video
npm run dev
```

### Add to Existing React Project
```bash
npm install remotion @remotion/cli
```

### Development Server (Remotion Studio)
```bash
npx remotion studio
# or
npm run dev
```

---

## Essential APIs

| API | Purpose | Example |
|-----|---------|---------|
| `useCurrentFrame()` | Get current frame (0-indexed) | `const frame = useCurrentFrame();` |
| `useVideoConfig()` | Get fps, width, height, duration | `const { fps, durationInFrames } = useVideoConfig();` |
| `interpolate()` | Map frame range to value range | `interpolate(frame, [0, 30], [0, 1])` |
| `spring()` | Physics-based animation | `spring({ frame, fps, config: { damping: 10 } })` |
| `<Sequence>` | Time-shift children | `<Sequence from={30}>...</Sequence>` |
| `<Series>` | Sequential clips | `<Series><Series.Sequence>...</Series.Sequence></Series>` |
| `<AbsoluteFill>` | Full-frame container | `<AbsoluteFill style={{ backgroundColor: '#000' }}>` |
| `<Img>` | Optimized image | `<Img src={staticFile('logo.png')} />` |
| `<Video>` | Embed video | `<Video src={staticFile('clip.mp4')} />` |
| `<Audio>` | Embed audio | `<Audio src={staticFile('music.mp3')} />` |
| `staticFile()` | Reference public/ files | `staticFile('assets/image.png')` |

See [references/api-reference.md](references/api-reference.md) for complete API documentation.

---

## Animation Patterns

### Fade In
```tsx
const frame = useCurrentFrame();
const opacity = interpolate(frame, [0, 30], [0, 1], {
  extrapolateRight: 'clamp',
});
return <div style={{ opacity }}>Content</div>;
```

### Spring Scale
```tsx
const frame = useCurrentFrame();
const { fps } = useVideoConfig();
const scale = spring({ frame, fps, config: { damping: 12, stiffness: 200 } });
return <div style={{ transform: `scale(${scale})` }}>Content</div>;
```

### Slide In From Left
```tsx
const frame = useCurrentFrame();
const x = interpolate(frame, [0, 30], [-100, 0], {
  extrapolateRight: 'clamp',
});
return <div style={{ transform: `translateX(${x}%)` }}>Content</div>;
```

### Staggered Elements
```tsx
const items = ['One', 'Two', 'Three'];
return (
  <AbsoluteFill>
    {items.map((item, i) => (
      <Sequence key={item} from={i * 15}>
        <FadeIn>{item}</FadeIn>
      </Sequence>
    ))}
  </AbsoluteFill>
);
```

### Typewriter Effect
```tsx
const frame = useCurrentFrame();
const text = "Hello, World!";
const chars = Math.floor(interpolate(frame, [0, 60], [0, text.length], {
  extrapolateRight: 'clamp',
}));
return <span>{text.slice(0, chars)}</span>;
```

See [references/animation-patterns.md](references/animation-patterns.md) for more patterns.

---

## Common Video Structures

### Basic Composition with Scenes
```tsx
// MyVideo.tsx
import { AbsoluteFill, Sequence } from 'remotion';
import { Intro } from './Intro';
import { MainContent } from './MainContent';
import { Outro } from './Outro';

export const MyVideo: React.FC = () => {
  return (
    <AbsoluteFill style={{ backgroundColor: '#0a0a0a' }}>
      <Sequence from={0} durationInFrames={90}>
        <Intro />
      </Sequence>
      <Sequence from={90} durationInFrames={180}>
        <MainContent />
      </Sequence>
      <Sequence from={270}>
        <Outro />
      </Sequence>
    </AbsoluteFill>
  );
};
```

### Registration (Root.tsx)
```tsx
import { Composition } from 'remotion';
import { MyVideo } from './MyVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="MyVideo"
        component={MyVideo}
        durationInFrames={300}
        fps={30}
        width={1920}
        height={1080}
      />
      <Composition
        id="MyVideoSquare"
        component={MyVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1080}
      />
    </>
  );
};
```

### Using Props for Templates
```tsx
type VideoProps = {
  title: string;
  backgroundColor: string;
};

export const TemplatedVideo: React.FC<VideoProps> = ({ title, backgroundColor }) => {
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <h1>{title}</h1>
    </AbsoluteFill>
  );
};

// In Root.tsx
<Composition
  id="TemplatedVideo"
  component={TemplatedVideo}
  durationInFrames={150}
  fps={30}
  width={1920}
  height={1080}
  defaultProps={{
    title: 'Default Title',
    backgroundColor: '#000',
  }}
/>
```

### Loading Async Data
```tsx
import { delayRender, continueRender } from 'remotion';
import { useEffect, useState } from 'react';

export const DataDrivenVideo: React.FC = () => {
  const [data, setData] = useState(null);
  const [handle] = useState(() => delayRender('Loading data'));

  useEffect(() => {
    fetch('/api/data')
      .then((res) => res.json())
      .then((json) => {
        setData(json);
        continueRender(handle);
      });
  }, [handle]);

  if (!data) return null;
  return <div>{data.content}</div>;
};
```

---

## Rendering Commands

### Render to MP4
```bash
npx remotion render src/index.ts MyVideo out/video.mp4
```

### Render with Custom Settings
```bash
npx remotion render src/index.ts MyVideo out/video.mp4 \
  --codec=h264 \
  --crf=18 \
  --scale=1
```

### Render WebM (VP8/VP9)
```bash
npx remotion render src/index.ts MyVideo out/video.webm --codec=vp8
```

### Render GIF
```bash
npx remotion render src/index.ts MyVideo out/video.gif --codec=gif
```

### Render Still Image
```bash
npx remotion still src/index.ts MyVideo out/thumbnail.png --frame=0
```

### Render Image Sequence
```bash
npx remotion render src/index.ts MyVideo out/frames/ --image-format=png --sequence
```

### Render with Props
```bash
npx remotion render src/index.ts TemplatedVideo out/video.mp4 \
  --props='{"title":"Custom Title","backgroundColor":"#1a1a2e"}'
```

### Render Specific Frame Range (Testing)
```bash
npx remotion render src/index.ts MyVideo test.mp4 --frames=0-30
```

### Parallel Rendering (Faster)
```bash
npx remotion render src/index.ts MyVideo out/video.mp4 --concurrency=8
```

---

## Design Guidance

### Frame Rates
- **30fps**: Standard video, social media
- **60fps**: Smooth motion, gaming content
- **24fps**: Cinematic feel

### Resolutions
| Format | Width | Height | Use Case |
|--------|-------|--------|----------|
| 1080p | 1920 | 1080 | YouTube, general |
| 4K | 3840 | 2160 | High quality |
| Square | 1080 | 1080 | Instagram |
| Portrait | 1080 | 1920 | TikTok, Reels |
| Twitter | 1280 | 720 | Twitter video |

### Best Practices
- Use `AbsoluteFill` for layering elements
- Keep components pure (no side effects based on frame)
- Preload fonts with `@remotion/google-fonts` or explicit loading
- Use `staticFile()` for all assets in `public/`
- Use `OffthreadVideo` for memory-efficient video embedding
- Test at lower resolution first, then render at full quality

### Typography
- Use web-safe fonts or load explicitly
- Scale font sizes relative to composition dimensions
- Account for safe zones (10% margins for TV/broadcast)

---

## QA Process

### Preview in Studio
```bash
npm run dev
# Opens Remotion Studio at http://localhost:3000
```

### List All Compositions
```bash
npx remotion compositions src/index.ts
```

### Validate Before Full Render
```bash
# Render first 30 frames only
npx remotion render src/index.ts MyVideo test.mp4 --frames=0-30

# Render at lower scale
npx remotion render src/index.ts MyVideo test.mp4 --scale=0.5
```

### Check Audio Sync
- Ensure all media has consistent fps
- Use `startFrom` and `endAt` props on `<Video>` and `<Audio>` for trimming
- Preview full video in Studio before rendering

---

## Common Use Cases

| Use Case | Key Components |
|----------|----------------|
| Product demo | `<Video>`, `<Sequence>`, text overlays |
| Social media | Square/portrait compositions, quick animations |
| Data viz | `interpolate()` for animated charts |
| Title sequence | Spring animations, staggered text |
| Slideshow | `<Series>`, `<Img>`, transitions |
| Tutorial | Screen recordings with callouts |
| Music visualizer | Audio analysis, reactive animations |

---

## Project Structure

```
my-video/
├── src/
│   ├── index.ts          # Entry point
│   ├── Root.tsx          # Composition registrations
│   ├── MyVideo.tsx       # Main composition
│   └── components/       # Reusable components
│       ├── Intro.tsx
│       ├── Title.tsx
│       └── animations/
├── public/               # Static assets (use staticFile())
│   ├── logo.png
│   └── music.mp3
├── remotion.config.ts    # Remotion configuration
└── package.json
```

---

## Next Steps

- See [references/api-reference.md](references/api-reference.md) for complete API documentation
- See [references/animation-patterns.md](references/animation-patterns.md) for animation recipes
- See [references/troubleshooting.md](references/troubleshooting.md) for common issues
