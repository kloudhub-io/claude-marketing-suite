---
name: create-vision-film
description: Use when the user wants to build a short cinematic brand film (~20 seconds) that conveys the product's emotional thesis rather than its mechanics. Triggers on "/create-vision-film", "create vision film", "make a brand video", "build a cinematic intro", "create a teaser for [product]". Different from create-feature-demo — vision films are emotional/abstract (problem → realization → answer → brand), no UI mockups, no playback bar, autoplay + loop. Requires marketing-brief.md from /setup-marketing-suite.
allowed-tools: Read, Glob, Grep, Write, Bash
---

# Create Vision Film

Build a 20-second cinematic brand film. Different beast from the feature demo:

| | Feature demo | Vision film |
|---|---|---|
| Length | ~60s | ~20s |
| Backdrop | White or product-themed | Dark (matches "ad-style" pieces) |
| Content | Real product UI | Emotional metaphor + brand reveal |
| Playback bar | Visible | Hidden |
| Autoplay | No | Yes + loop |
| Audio | VO or cues | Sparse cues + maybe drone |
| Captions | Multiple per scene | Few, punchy, large type |

## Prerequisites

Read `marketing-brief.md`. If missing, direct user to `/setup-marketing-suite`.

## Structure — the four beats

<beats>
1. **Reel** (0–5s) — show the problem viscerally. Anything that establishes the scale of the bad thing the product fixes. For a privacy app: a chat thread accumulating sensitive content. For a productivity app: an overflowing inbox. For a fintech: a frozen account screen.

2. **Hammer** (5–11s) — a punchy headline appears in massive type, dwelling. Usually 1–3 words. Example: "Still there." / "Forever." / "Until now."

3. **Dissolve / transform** (11–15s) — the problem vanishes / transforms. Particle scatter, dissolve into white, a folding-away effect. This is the answer arriving.

4. **Brand** (15–20s) — logo + wordmark + tagline. Hold on it. Loop.
</beats>

## Tech setup

Same architecture as the feature demo (see `create-feature-demo` references), but different `Stage` config:

```jsx
<Stage
  width={1920}
  height={1080}
  duration={20}
  background="#07080A"            /* dark — matches the dark-mode aesthetic */
  autoplay={true}                 /* play immediately */
  loop={true}                     /* loop forever; no record button */
  hideBar={true}                  /* no playback bar */
  playbackSpeed={1.0}
>
  <Scene_Reel/>
  <Scene_Hammer/>
  <Scene_Dissolve/>
  <Scene_Brand/>
</Stage>
```

## Visual treatment

<treatment>
- **Backdrop:** dark radial gradient `radial-gradient(ellipse at 50% 35%, #16171B 0%, #0B0C0F 60%, #07080A 100%)`
- **Type:** large, confident, bright white. 120–180px for hammer beats, 60–80px for supporting lines.
- **Motion:** ease curves only. No bounces or springs. Soft fades, scale-and-fade, x/y translates.
- **Particle dissolve:** for the transform beat. ~100–150 particles scattering outward from a center point. Color shifts from foreground to background as they scatter.
- **Logo treatment in close:** logo in a brand-color circle, wordmark below, tagline below that. Subtle scale-in + fade-in.
</treatment>

## Audio

Sparse — this is a 20s piece. Recommended cues:

- `0.0s` — `drone.mp3` (sustained low pad, 6–8s loopable, sets unease)
- `5.0s` — `hammer.mp3` (one-shot impact when the headline lands)
- `11.0s` — `dissolve.mp3` (the particle scatter)
- `15.0s` — `stinger.mp3` (logo reveal chime)

No voiceover. The captions and visuals carry it.

## Anti-AI rules (extra strict for vision films)

Vision films are emotional — they live or die on craft. Even more important than for demos:

1. **One focal element per beat.** Hammer beat is JUST the words. No supporting decoration.
2. **No abstract iconography.** Either use real-looking content (chat bubble with text, document with date stamp) or pure type. Don't render a "lock" or "clock" as a generic icon.
3. **No flashy transitions.** Cuts and fades only. No swooshes, wipes, or stinger transitions (other than the audio).
4. **Hold beats long enough to land.** A 1-second hammer beat is rushed. Give it 4+ seconds.
5. **Dark is harder than light.** Every element on a dark background needs more care — contrast, glow, weight. Don't put low-contrast type on the gradient.

## Output

Single self-contained file: `vision-film.html` in `[product-name]-marketing-suite/`.

Same recording machinery as the feature demo, but hidden by default (autoplay/loop is for the in-page experience; recording is for export).

## Workflow

1. Read `marketing-brief.md`.
2. Propose the four beats specifically tailored to the product. Get user confirmation.
3. Build the HTML.
4. Show user. Iterate on the hammer beat first — that line is what people remember.
