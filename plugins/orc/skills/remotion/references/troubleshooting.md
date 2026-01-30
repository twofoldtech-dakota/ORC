# Remotion Troubleshooting

Common issues and their solutions.

---

## Installation Issues

### "Cannot find module 'remotion'"

**Cause:** Remotion packages not installed.

**Solution:**
```bash
npm install remotion @remotion/cli @remotion/player
```

### "Cannot find module '@remotion/bundler'"

**Cause:** Missing bundler dependency.

**Solution:**
```bash
npm install @remotion/bundler
```

### FFMPEG not found / Codec errors

**Cause:** FFMPEG not available on system.

**Solution:**
```bash
# Remotion can install FFMPEG automatically
npx remotion install ffmpeg

# Or install system-wide
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt install ffmpeg

# Windows (via Chocolatey)
choco install ffmpeg
```

### Node version incompatibility

**Cause:** Remotion requires Node.js 16+.

**Solution:**
```bash
node --version  # Check version
nvm install 18  # Install Node 18
nvm use 18      # Switch to Node 18
```

---

## Context Errors

### "useCurrentFrame must be used inside a Remotion context"

**Cause:** Hook called outside of a Composition or Player context.

**Wrong:**
```tsx
// Component used directly without Remotion context
const App = () => {
  const frame = useCurrentFrame(); // Error!
  return <div>{frame}</div>;
};
```

**Correct:**
```tsx
// In Root.tsx - register with Composition
<Composition
  id="MyVideo"
  component={MyComponent}
  durationInFrames={300}
  fps={30}
  width={1920}
  height={1080}
/>

// Or wrap in Player for preview
import { Player } from '@remotion/player';

<Player
  component={MyComponent}
  durationInFrames={300}
  fps={30}
  compositionWidth={1920}
  compositionHeight={1080}
/>
```

### "useVideoConfig must be used inside a Remotion context"

**Same cause and solution as above.**

---

## Rendering Issues

### Blank video / Nothing rendering

**Possible causes:**

1. **Component returns null at frame 0:**
```tsx
// Wrong
const MyComponent = () => {
  const [data, setData] = useState(null);
  if (!data) return null; // Returns null before data loads
};

// Correct - use delayRender
const MyComponent = () => {
  const [data, setData] = useState(null);
  const [handle] = useState(() => delayRender('Loading'));

  useEffect(() => {
    loadData().then((d) => {
      setData(d);
      continueRender(handle);
    });
  }, []);

  if (!data) return null;
  return <div>{data}</div>;
};
```

2. **Zero opacity at start:**
```tsx
// Check your interpolate ranges
const opacity = interpolate(frame, [0, 30], [0, 1]); // Starts at 0
```

3. **Element positioned off-screen:**
```tsx
// Check transforms
const x = interpolate(frame, [0, 30], [-200, 0]); // Starts off-screen
```

**Debug steps:**
```tsx
// Add visible element to verify rendering
<AbsoluteFill style={{ backgroundColor: 'red' }}>
  <div style={{ color: 'white', fontSize: 48 }}>
    Frame: {useCurrentFrame()}
  </div>
</AbsoluteFill>
```

### Black frames at start or end

**Cause:** Sequence timing doesn't cover full duration.

**Wrong:**
```tsx
<Composition durationInFrames={300} ...>
  {/* Nothing visible at frame 0-29 */}
  <Sequence from={30}>
    <Content />
  </Sequence>
</Composition>
```

**Correct:**
```tsx
<Composition durationInFrames={300} ...>
  <Sequence from={0} durationInFrames={30}>
    <Intro />
  </Sequence>
  <Sequence from={30}>
    <Content />
  </Sequence>
</Composition>
```

### Render taking too long

**Solutions:**

1. **Increase concurrency:**
```bash
npx remotion render src/index.ts MyVideo out.mp4 --concurrency=8
```

2. **Test at lower resolution:**
```bash
npx remotion render src/index.ts MyVideo out.mp4 --scale=0.5
```

3. **Render frame range for testing:**
```bash
npx remotion render src/index.ts MyVideo out.mp4 --frames=0-60
```

4. **Optimize assets:**
   - Reduce image sizes
   - Use compressed videos
   - Avoid unnecessary high-resolution assets

5. **Check for re-renders:**
```tsx
// Memoize expensive components
const ExpensiveComponent = React.memo(({ data }) => {
  // ...
});
```

### Out of memory during render

**Solutions:**

1. **Reduce concurrency:**
```bash
npx remotion render src/index.ts MyVideo out.mp4 --concurrency=2
```

2. **Use OffthreadVideo for video embedding:**
```tsx
// Instead of
<Video src={staticFile('video.mp4')} />

// Use
<OffthreadVideo src={staticFile('video.mp4')} />
```

3. **Reduce frame buffer:**
```bash
npx remotion render src/index.ts MyVideo out.mp4 --frames-per-lambda=10
```

### Colors look wrong in output

**Cause:** Color space mismatch.

**Solution (in remotion.config.ts):**
```ts
import { Config } from '@remotion/cli/config';
Config.setColorSpace('bt709');
```

---

## Media Issues

### Video stuttering in preview (Studio)

**Cause:** Heavy video files being decoded in real-time.

**Solution:**
```tsx
// Use OffthreadVideo for smoother preview
import { OffthreadVideo } from 'remotion';

<OffthreadVideo src={staticFile('heavy-video.mp4')} />
```

### Audio/Video not loading

**Cause:** Wrong path or missing staticFile().

**Wrong:**
```tsx
<Video src="/public/video.mp4" />
<Video src="./video.mp4" />
```

**Correct:**
```tsx
import { Video, staticFile } from 'remotion';

<Video src={staticFile('video.mp4')} />
```

**File structure:**
```
project/
├── public/
│   └── video.mp4  ← Put media here
└── src/
```

### Audio/video sync issues

**Causes and solutions:**

1. **Mixed frame rates:**
   - Ensure all videos have consistent fps
   - Convert videos to match composition fps

2. **Wrong playback rate:**
```tsx
<Video
  src={staticFile('video.mp4')}
  playbackRate={1}  // Ensure this is 1
/>
```

3. **Trim issues:**
```tsx
<Video
  src={staticFile('video.mp4')}
  startFrom={0}   // Frames, not seconds
  endAt={150}     // Frames, not seconds
/>
```

### Fonts not loading

**Solution 1: Use @remotion/google-fonts**
```bash
npm install @remotion/google-fonts
```

```tsx
import { loadFont } from '@remotion/google-fonts/Inter';

const { fontFamily } = loadFont();

<div style={{ fontFamily }}>Text</div>
```

**Solution 2: Load explicitly with delayRender**
```tsx
import { delayRender, continueRender } from 'remotion';

const [handle] = useState(() => delayRender('Loading fonts'));

useEffect(() => {
  document.fonts.load('16px "Custom Font"').then(() => {
    continueRender(handle);
  });
}, [handle]);
```

**Solution 3: Include in public folder**
```
public/
└── fonts/
    └── CustomFont.woff2
```

```css
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/CustomFont.woff2') format('woff2');
}
```

---

## Common Mistakes

### Forgetting extrapolateRight: 'clamp'

**Problem:** Animation continues beyond intended range.

```tsx
// Wrong - opacity goes above 1 after frame 30
const opacity = interpolate(frame, [0, 30], [0, 1]);

// At frame 60, opacity = 2 (extrapolated)
```

**Correct:**
```tsx
const opacity = interpolate(frame, [0, 30], [0, 1], {
  extrapolateRight: 'clamp',
});
// At frame 60, opacity = 1 (clamped)
```

### Using useState for animation values

**Problem:** State doesn't sync with frame-based rendering.

```tsx
// Wrong
const [opacity, setOpacity] = useState(0);

useEffect(() => {
  setOpacity(frame / 30);  // Race conditions, not deterministic
}, [frame]);
```

**Correct:**
```tsx
// Derive directly from frame
const frame = useCurrentFrame();
const opacity = interpolate(frame, [0, 30], [0, 1]);
```

### Hardcoding fps in calculations

**Problem:** Breaks if composition fps changes.

```tsx
// Wrong
const seconds = frame / 30;  // Assumes 30fps

// Correct
const { fps } = useVideoConfig();
const seconds = frame / fps;
```

### Non-deterministic components

**Problem:** Random values change on every render.

```tsx
// Wrong - different value each render
const x = Math.random() * 100;

// Correct - deterministic random
import { random } from 'remotion';
const x = random('particle-x') * 100;  // Same value every render
```

### Async without delayRender

**Problem:** Component renders before data is loaded.

```tsx
// Wrong
const [data, setData] = useState(null);
useEffect(() => {
  fetch('/api/data').then(setData);
}, []);

// Correct
const [data, setData] = useState(null);
const [handle] = useState(() => delayRender('Fetching data'));

useEffect(() => {
  fetch('/api/data')
    .then((d) => {
      setData(d);
      continueRender(handle);
    })
    .catch(() => {
      continueRender(handle);  // Always continue, even on error
    });
}, [handle]);
```

---

## CLI Issues

### "No compositions found"

**Cause:** Entry file doesn't export compositions correctly.

**Check src/index.ts:**
```tsx
import { registerRoot } from 'remotion';
import { RemotionRoot } from './Root';

registerRoot(RemotionRoot);
```

**Check src/Root.tsx:**
```tsx
import { Composition } from 'remotion';
import { MyVideo } from './MyVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <Composition
      id="MyVideo"
      component={MyVideo}
      durationInFrames={300}
      fps={30}
      width={1920}
      height={1080}
    />
  );
};
```

### "Cannot read properties of undefined"

**Cause:** Props not passed or defaultProps not set.

**Solution:**
```tsx
<Composition
  id="MyVideo"
  component={MyVideo}
  durationInFrames={300}
  fps={30}
  width={1920}
  height={1080}
  defaultProps={{
    title: 'Default Title',  // Provide defaults
  }}
/>
```

### Command not found: remotion

**Cause:** Remotion CLI not in PATH.

**Solutions:**
```bash
# Use npx
npx remotion render ...

# Or install globally
npm install -g @remotion/cli
remotion render ...
```

---

## Performance Tips

### Optimize component renders

```tsx
// Memoize static components
const StaticElement = React.memo(() => (
  <div>This doesn't change</div>
));

// Use useMemo for expensive calculations
const particles = useMemo(() =>
  generateParticles(1000),
  []  // Empty deps = calculate once
);
```

### Optimize images

- Use appropriate sizes (don't load 4K images for small elements)
- Prefer WebP or AVIF formats
- Use `loading="eager"` for above-the-fold images

### Reduce composition complexity

- Split complex videos into multiple compositions
- Render and compose in post if needed
- Use `<OffthreadVideo>` for video-in-video scenarios

### Use appropriate codecs

```bash
# Fastest render (larger file)
npx remotion render ... --codec=h264 --crf=23

# Slower render (smaller file)
npx remotion render ... --codec=h264 --crf=18

# Fastest for web
npx remotion render ... --codec=vp8
```
