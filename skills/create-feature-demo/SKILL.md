---
name: create-feature-demo
description: Use when the user wants to build a 60-second animated product demo video that plays in the browser and is recordable to .webm. Triggers on "/create-feature-demo", "make a feature demo video", "build product walkthrough", "create demo video for [product]", "record a product demo". Requires marketing-brief.md from /setup-marketing-suite to exist. Builds a single self-contained HTML file with timeline-driven scene composition, real product UI mockups extracted from the codebase, captions, optional voiceover or sound cues, and an in-browser screen-recording flow that exports to .webm.
allowed-tools: Read, Glob, Grep, Write, Bash
---

# Create Feature Demo

Build a 60-second animated product demo as a single self-contained HTML file. The demo:

- Plays in the browser on `file://` (no localhost needed)
- Shows the actual product UI extracted from the codebase
- Has a record button that captures it to `.webm` via `getDisplayMedia` + `MediaRecorder`
- Optionally syncs to a voiceover MP3 OR fires discrete sound cues

## Prerequisites

Read `marketing-brief.md` first. If it doesn't exist, tell the user to run `/setup-marketing-suite` and stop.

The brief gives you: design tokens, screens, audience, audio choice, tagline.

## Architecture — build these primitives first in the HTML

See `references/architecture.md` for full primitive specs. Quick summary:

- **`Stage`** — 1920×1080 fixed canvas with timeline. Manages `time`, `playing`, `duration`, `playbackSpeed`, `recording`. Auto-fits viewport via CSS scale.
- **`Sprite`** — `<Sprite start={5} end={10}>` only renders during that window. Children receive `localTime` and `progress`.
- **`Camera`** — interpolates between `{ t, x, y, zoom, ease }` keyframes, applies CSS transform to children.
- **`TimelineContext`** — React context exposing the timeline state. `useTimeline()` hook.
- **`PlaybackBar`** — play/pause/scrub/restart/record. Hidden during recording.
- Easing helpers (hand-rolled): `easeInCubic`, `easeOutCubic`, `easeInOutCubic`, `easeOutQuart`, `easeOutBack`.

## Scene composition

Aim for 8–12 scenes spanning ~60 seconds. Each scene is a `<Sprite>` with `start` and `end`. The standard arc:

<scene_arc>
1. **Hook** (0–4s) — establish the problem viscerally
2. **Problem deepens** (4–14s) — show the scale/consequence
3. **Realization** (14–22s) — viewer arrives at the question the product answers
4. **Product reveal** (22–34s) — the actual app UI doing the thing
5. **Why it works** (34–46s) — 3 quick reasons / benefits
6. **The promise** (46–54s) — emotional payoff (file vanishes, success state)
7. **Close** (54–60s) — logo + tagline + URL
</scene_arc>

Adapt to product. A B2B tool might compress the emotional arc; a consumer app might extend it.

## Product UI mockups

Read the screens identified in `marketing-brief.md`. For each screen used in the demo, reproduce its layout in HTML/CSS using the design tokens from the brief.

Key components to build:
- **`PhoneShell`** — dark bezel `#16161A`, 16px padding, 44px outer-radius, 36px inner-radius. Light or dark inner-screen variant.
- **Status bar** — 9:41 time + signal/wifi/battery SVGs. Match light/dark variant to the screen.
- **Home indicator** — small bar at bottom, 4px tall, 100px wide.

Match the real screen layouts as closely as possible. Use the design tokens verbatim — don't invent new colors.

## Audio (pick one based on the brief)

### If brief says "voiceover":
- Add a `<VoiceOver src="voiceover.mp3"/>` component
- **Lock `playbackSpeed={1.0}`** — audio plays at natural rate; timeline must match
- Sync formula: `expected = time / playbackSpeed`. If `Math.abs(audio.currentTime - expected) > 0.35`, seek
- First-gesture unlock: show a "click anywhere to start" pill that disappears on first click
- Hide mute button when `recording === true`

### If brief says "sound cues":
- Add `<SoundCue src="sounds/[name].mp3" at={X}/>` components for 6–10 discrete moments
- Standard cues: `tap`, `whoosh`, `thud`, `chime`, `stinger`
- `playbackSpeed` can be > 1.0 (e.g., 1.3) since cues are short and one-shot

See `references/audio.md` for full specs.

## Recording flow

The record button triggers `getDisplayMedia({ video: { frameRate: 30, displaySurface: 'browser' }, audio: true })`. Key rules:

<recording_rules>
- **`audio: true` as a literal boolean** — this is what triggers the "Share tab audio" toggle. Do not pass an audio-constraints object; it suppresses the toggle.
- **Hide all UI chrome when `recording === true`** — playback bar, mute button, tap-to-start pill. They'd capture into the .webm otherwise.
- **Stop the timeline from looping when recording** — `loop && !recording` in the wrap check. Otherwise the .webm tail captures Scene 1.
- **Auto-stop timing:** `setTimeout(() => recorder.stop(), (duration / playbackSpeed + 0.2) * 1000)`. The 0.2s tail catches the final frame.
- **Codec:** prefer `video/webm;codecs=vp9,opus`, fall back to `video/webm;codecs=vp8,opus`. Bitrate: 8 Mbps video, 128 kbps audio.
</recording_rules>

See `references/recording.md` for the full recording implementation.

## Design discipline

These patterns make designs look AI-generated. Avoid them:

<anti_ai_rules>
1. **No emojis** in visuals or captions
2. **No decorative glows** unless they serve focus
3. **No concentric rings** around icons
4. **No random rotations** — every rotation should be deliberate (a stack at -3°/2°/-1° reads as "stacked"; a scatter at -12°/8°/-6°/10° reads as "AI tried to look creative")
5. **No abstract icons-as-content** (don't render OTP as giant monospace digits, statements as a giant `$`)
6. **No em-dashes** in body copy — use commas or "is"
7. **One strong focal element per scene** — not "logo + tagline + pill + chip + glow"
8. **Show real product UI** wherever possible, not metaphorical illustrations
9. **Match the product's existing visual identity** — don't invent a new design language
</anti_ai_rules>

## Captions

One sentence per beat, anchored to the bottom of the canvas. Style:

- Big confident type: Inter 800-weight, 76–96px, letter-spacing -0.02em
- Light backgrounds → ink color (`#0B0C0F`); dark → soft white (`rgba(255,255,255,0.92)`) with text-shadow
- Optional eyebrow above: smaller, uppercase, letter-spacing 0.10em, brand-color accent
- Each caption is its own nested `<Sprite>` with its own `start`/`end`

## Output file structure

```
[product-name]-marketing-suite/
  feature-demo.html              ← the demo (self-contained)
  voiceover.mp3                  ← optional, user-provided
  sounds/                        ← optional, user-provided
    tap.mp3
    whoosh.mp3
    stinger.mp3
```

## Workflow

1. Read `marketing-brief.md`. If missing, stop and direct user to `/setup-marketing-suite`.
2. Read 3–5 of the primary screens from the codebase to understand the UI in detail.
3. Propose the scene breakdown as a numbered list (one phrase per scene). Get user confirmation.
4. Build the HTML: infrastructure → product UI components → scene composition → captions → audio.
5. Tell the user where to drop their voiceover.mp3 / sound files.
6. Recommend they record one .webm to verify everything works end-to-end.

## What to NOT do

- Do not skip reading the brief
- Do not invent design tokens — use what's in the brief
- Do not write 12 scenes before showing the user. Show after scene 4.
- Do not use a music bed unless the user explicitly asks. Sound cues > music for product demos.
- Do not exceed 60s without confirming. Each extra second has to be earned.
