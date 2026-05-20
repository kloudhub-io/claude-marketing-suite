# Particle Dissolve — Reference

The transform/dissolve beat in a vision film typically uses a particle scatter. Here's the component.

## Component

```jsx
function ParticleDissolve({ x, y, w = 280, h = 180, startAt = 0, duration = 1.4, count = 110, fromColor = '#0B0C0F', toColor = '#FFFFFF' }) {
  const { localTime } = useSprite();
  const t = clamp((localTime - startAt) / duration, 0, 1);
  if (t <= 0) return null;

  const particles = React.useMemo(() => Array.from({ length: count }).map((_, i) => {
    const seed = (i * 9301 + 49297) % 233280;
    const r = (seed / 233280);
    const angle = r * Math.PI * 2;
    const dist = 200 + Math.random() * 300;
    return {
      sx: (Math.random() - 0.5) * w,
      sy: (Math.random() - 0.5) * h,
      dx: Math.cos(angle) * dist,
      dy: Math.sin(angle) * dist - 80, // slight upward bias for "rising" feel
      size: 3 + Math.random() * 6,
      delay: Math.random() * 0.4,
    };
  }), [count, w, h]);

  return (
    <div style={{
      position: 'absolute',
      left: x, top: y,
      transform: 'translate(-50%, -50%)',
      pointerEvents: 'none',
    }}>
      {particles.map((p, i) => {
        const pt = clamp((t - p.delay) / (1 - p.delay), 0, 1);
        const eased = easeOut(pt);
        const tx = p.sx + p.dx * eased;
        const ty = p.sy + p.dy * eased;
        const op = 1 - pt;
        return (
          <div key={i} style={{
            position: 'absolute',
            left: 0, top: 0,
            width: p.size, height: p.size, borderRadius: 99,
            background: lerpColor(fromColor, toColor, pt),
            transform: `translate(${tx}px, ${ty}px)`,
            opacity: op,
          }}/>
        );
      })}
    </div>
  );
}

function lerpColor(a, b, t) {
  // Hex string → RGB lerp → rgb() string
  const ah = parseInt(a.slice(1), 16);
  const bh = parseInt(b.slice(1), 16);
  const ar = (ah >> 16) & 255, ag = (ah >> 8) & 255, ab = ah & 255;
  const br = (bh >> 16) & 255, bg = (bh >> 8) & 255, bb = bh & 255;
  const r = Math.round(ar + (br - ar) * t);
  const g = Math.round(ag + (bg - ag) * t);
  const bl = Math.round(ab + (bb - ab) * t);
  return `rgb(${r},${g},${bl})`;
}
```

## Usage

```jsx
<ParticleDissolve
  x={CENTER_X}
  y={CENTER_Y}
  w={300} h={200}
  startAt={11.0}     /* relative to parent <Sprite> */
  duration={1.6}
  count={140}
  fromColor="#FFFFFF"   /* particles start white (matching the document) */
  toColor="#0B0C0F"     /* fade to dark (matching the backdrop) */
/>
```

## Tuning

- **`count`** — 80 for subtle, 150+ for dense scatter. More particles = heavier render but smoother visual.
- **`duration`** — 1.2s feels quick; 1.8s feels deliberate. Match the rhythm of the rest of the beat.
- **`dist`** in particle init — controls spread radius. Larger = more dramatic.
- **`dy` upward bias** — `-80` gives a "rising up" feel. Remove for pure radial scatter.

## Multiple dissolves at once

For "everything vanishes simultaneously" moments (e.g., all files in a grid dissolving), instantiate one `<ParticleDissolve>` per location with slightly staggered `startAt` values for a rolling-wave effect:

```jsx
{files.map((f, i) => (
  <React.Fragment key={i}>
    <FileTile x={f.x} y={f.y} opacity={fileFadeOut[i]} />
    <ParticleDissolve x={f.x} y={f.y} startAt={1.0 + i * 0.15} />
  </React.Fragment>
))}
```
