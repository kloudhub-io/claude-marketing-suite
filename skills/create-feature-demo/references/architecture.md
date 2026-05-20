# Animation Architecture — Reference

The animated assets (feature demo, vision film) share this core architecture. Built into every asset HTML.

## Top-level dependencies (CDN, inlined)

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js" crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" crossorigin="anonymous"></script>
```

All JSX goes inside `<script type="text/babel">` blocks. No build step.

## Easing helpers

Hand-rolled, no libraries:

```js
const Easing = {
  linear: (t) => t,
  easeInCubic:    (t) => t * t * t,
  easeOutCubic:   (t) => (--t) * t * t + 1,
  easeInOutCubic: (t) => (t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1),
  easeOutQuart:   (t) => 1 - (--t) * t * t * t,
  easeInOutQuart: (t) => (t < 0.5 ? 8 * t * t * t * t : 1 - 8 * (--t) * t * t * t),
  easeOutExpo:    (t) => (t === 1 ? 1 : 1 - Math.pow(2, -10 * t)),
  easeOutBack:    (t) => {
    const c1 = 1.70158, c3 = c1 + 1;
    return 1 + c3 * Math.pow(t - 1, 3) + c1 * Math.pow(t - 1, 2);
  },
};

const clamp = (v, mn, mx) => Math.max(mn, Math.min(mx, v));
const lerp  = (a, b, t) => a + (b - a) * t;
```

## TimelineContext + Sprite

```js
const TimelineContext = React.createContext({
  time: 0, duration: 60, playing: false, recording: false
});
const useTimeline = () => React.useContext(TimelineContext);

const SpriteContext = React.createContext({ localTime: 0, progress: 0, duration: 0 });
const useSprite = () => React.useContext(SpriteContext);

function Sprite({ start = 0, end = Infinity, children, keepMounted = false }) {
  const { time } = useTimeline();
  const visible = time >= start && time <= end;
  if (!visible && !keepMounted) return null;
  const duration = end - start;
  const localTime = Math.max(0, time - start);
  const progress = duration > 0 && isFinite(duration)
    ? clamp(localTime / duration, 0, 1)
    : 0;
  const value = { localTime, progress, duration, visible };
  return (
    <SpriteContext.Provider value={value}>
      {typeof children === 'function' ? children(value) : children}
    </SpriteContext.Provider>
  );
}
```

## Stage (the timeline owner)

Key responsibilities:
1. Owns the timeline state (`time`, `playing`, `duration`, `playbackSpeed`, `recording`)
2. Auto-fits canvas to viewport via CSS transform
3. Hides UI chrome (playback bar, mute button) when `recording === true`
4. Implements the animation loop with `playbackSpeed` multiplier
5. **Stops looping when recording** (critical — otherwise the .webm tail loops back to scene 1)

The critical animation loop:

```js
React.useEffect(() => {
  if (!playing) { lastTsRef.current = null; return; }
  const step = (ts) => {
    if (lastTsRef.current == null) lastTsRef.current = ts;
    const dt = (ts - lastTsRef.current) / 1000 * playbackSpeed;
    lastTsRef.current = ts;
    setTime((t) => {
      let next = t + dt;
      if (next >= duration) {
        // Critical: only loop if NOT recording
        if (loop && !recording) next = next % duration;
        else { next = duration; setPlaying(false); }
      }
      return next;
    });
    rafRef.current = requestAnimationFrame(step);
  };
  rafRef.current = requestAnimationFrame(step);
  return () => {
    if (rafRef.current) cancelAnimationFrame(rafRef.current);
    lastTsRef.current = null;
  };
}, [playing, duration, loop, recording, playbackSpeed]);
```

## Camera (keyframed pan/zoom)

```js
function Camera({ keyframes, children, stageW = 1920, stageH = 1080 }) {
  const { time } = useTimeline();
  // Find current keyframe pair
  // Interpolate x, y, zoom between them using the specified ease
  // Apply as CSS transform: translate(-x, -y) scale(zoom) translate(stageW/2, stageH/2)
  // (Center the zoom point on the target.)
  // ...
}
```

Keyframe shape:
```js
const camKf = [
  { t: 0.0, x: 960,  y: 540, zoom: 1.0 },
  { t: 2.0, x: 1200, y: 800, zoom: 1.4, ease: Easing.easeInOutCubic },
  { t: 4.0, x: 600,  y: 400, zoom: 1.2, ease: Easing.easeOutQuart },
];
```

## PlaybackBar

A dark pill at the bottom-center with: restart, play/pause, time `0:00 / 1:00`, scrubbable track, optional download button. Hidden during recording via `{!recording && <PlaybackBar .../>}`.

## Keyboard shortcuts (standard)

- `Space` — play/pause
- `←` / `→` — seek 1s (shift = 5s)
- `Home` or `0` — restart

## Composition pattern

```jsx
function App() {
  return (
    <Stage width={1920} height={1080} duration={60} background="#FFFFFF" playbackSpeed={1.0}>
      <Scene_Hook/>
      <Scene_Problem/>
      <Scene_Realization/>
      <Scene_Product/>
      <Scene_WhyItWorks/>
      <Scene_Promise/>
      <Scene_Close/>
      <SoundTrack/>
      {/* Optional: <VoiceOver src="voiceover.mp3"/> */}
    </Stage>
  );
}
```

Each `Scene_*` is a React function returning a `<Sprite start={X} end={Y}>` with the scene's content.
