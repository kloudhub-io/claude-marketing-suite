# Prompt: build an in-browser product demo video

Paste the block below into a fresh Claude conversation, replacing the bracketed placeholders with your specifics. The prompt captures the tech stack, architecture, design discipline, and gotchas from the Fliko demo build.

---

## THE PROMPT

I want you to build a product demo video for **[PRODUCT_NAME]** — [ONE_SENTENCE_DESCRIPTION].

The deliverable is a **single self-contained HTML file** that plays a fully animated product demo in the browser, with a button to record it as a downloadable `.webm` video.

### Tech stack — non-negotiable

- **One HTML file**, no build step, no Node/Vite/Webpack
- React 18 + Babel-Standalone via unpkg CDN
- All `.jsx` modules inlined as `<script type="text/babel">` blocks
- Must play on `file://` (no local server)
- Inter font loaded from Google Fonts
- Pure HTML/CSS/SVG for product mockups (no React Native, no canvas drawing libraries)

### Architecture — build these primitives first

- **`Stage`** — fixed canvas (1920×1080). Manages `time`, `playing`, `duration`, `recording`, `playbackSpeed`. Auto-fits to viewport via CSS transform. Loops by default; **stops looping when `recording` is true** (otherwise the .webm tail captures Scene 1 again).
- **`Sprite`** — `<Sprite start={5} end={10}>` only renders when timeline is in that window. Children receive `localTime` and `progress` via render-prop or context.
- **`Camera`** — interpolates between keyframes (`{ t, x, y, zoom, ease }`) and applies CSS transform to children. Used to pan/zoom across the canvas.
- **`TimelineContext`** — React context exposing the full timeline state. `useTimeline()` hook for descendants.
- **Easing helpers** — hand-roll `easeInCubic`, `easeOutCubic`, `easeInOutCubic`, `easeOutQuart`, `easeOutBack`. No animation libraries.
- **`PlaybackBar`** — play/pause, restart, scrubbable track, time display, download button. **Hide entirely when recording.**
- **Helpers** — `clamp(v, min, max)`, `lerp(a, b, t)`, `bezierPoint(...)` for cursor paths.

### Recording (download as .webm)

- Button calls `navigator.mediaDevices.getDisplayMedia({ video: { frameRate: 30, displaySurface: 'browser' }, audio: true })`. **`audio: true` as a boolean is what makes the "Share tab audio" toggle appear in the share dialog. Don't pass an audio-constraints object — it hides the toggle.**
- `MediaRecorder` with `video/webm;codecs=vp9,opus`, video bitrate 8 Mbps, audio bitrate 128 kbps.
- Auto-stop at `(duration / playbackSpeed + 0.2) * 1000` ms after `recorder.start()`. The 0.2s tail catches the final frame.
- When recording starts: hide playback bar, hide any mute buttons, hide any tap-to-start pills. They'd otherwise capture into the .webm.

### Audio (pick one approach, not both)

**Option A: Voiceover.** A `<VoiceOver src="voiceover.mp3"/>` component:
- Plays in sync with the timeline
- Drift formula: `expected = time / playbackSpeed` (audio plays at natural 1.0× rate regardless of playbackSpeed). If `Math.abs(audio.currentTime - expected) > 0.35`, seek.
- First-gesture unlock: browsers gate autoplay. Show a "click anywhere" pill that disappears on first click.
- Mute button hidden during recording.
- **If using VO, set `playbackSpeed={1.0}`** — audio doesn't speed up with the timeline.

**Option B: Discrete sound cues.** A `<SoundCue src="path.mp3" at={2.5}/>` component that fires once when the timeline crosses `at`. 6–10 cues throughout. Cleaner than continuous music. Used by Apple, Linear, Vercel in their product videos.

For consumer homepage videos: **lean toward sound cues**. Most viewers see autoplay-muted first; sound cues degrade gracefully to silence. VO requires unmute.

### Scenes — compose 6–14 of them

Each scene is a `<Sprite start={X} end={Y}>` containing:
- A `Camera` with keyframes panning/zooming around the canvas
- One or more product mockups (phone shells, browser frames)
- Captions appearing/disappearing via nested Sprites
- Optional: anchored cursors, particle dissolves, spotlights

Target total runtime: **30s for social, 60s for canonical, 90s if there's a real product story**.

### Product UI mockups

Build these in HTML/CSS:
- **`PhoneShell`** — dark bezel (#16161A), 44px padding, 88px outer-radius, 72px inner-radius, home indicator at bottom. Light/dark inner-screen variant.
- **Status bar** — 9:41 + signal/wifi/battery SVGs. Inline SVG, two variants (light/dark text).
- **Screen-specific UIs** — port from the real codebase where possible. Match the real design tokens (colors, weights, spacings). If no codebase access, approximate.

If the real app has design tokens, **lift them verbatim**: color palette, font weights, border-radii, spacing scale.

### Captions

- One sentence per beat. Land at the bottom of the canvas.
- Style: Inter 800-weight, 60–90px, letter-spacing -0.02em
- Color: dark text on light backgrounds (`#0B0C0F`) / soft-white on dark (`rgba(255,255,255,0.92)` + text-shadow)
- Optional eyebrow above (smaller, uppercase, letter-spaced 0.10em)
- Stagger captions throughout the scene — don't dump three lines at once

### Design discipline — anti-AI-template rules

These are the patterns that make designs look "AI-generated." Avoid them:

1. **No emojis** in visuals or captions. Use SVG icons.
2. **No decorative glows** unless they serve focus. A glow under a CTA is fine; a glow under a generic icon is template-stock.
3. **No concentric rings** around icons. One ring max.
4. **No random rotations** — every rotation should be deliberate. A stack at -3°/2°/-1° reads as "stacked." A scatter at -12°/8°/-6°/10°/-2° reads as "AI tried to look creative."
5. **No abstract icons-as-content.** Don't render an OTP as giant monospace digits or a bank statement as a giant `$` sign. Render the actual document layout — header, table rows, signature.
6. **No em-dashes** in body copy. Use commas, periods, or "is."
7. **One strong focal element per scene.** Not "logo + tagline + send-pill + 10-second-chip + glow." Pick one.
8. **Show real product UI** wherever possible, not metaphorical illustrations. A "set a timer" beat shows the actual timer card from your app, not an abstract clock.
9. **Don't tilt the camera unless there's a reason.** Stationary framing reads as confident; constant pans read as nervous.

### Gotchas — read these before writing code

1. **`file://` + canvas + cross-origin images = tainted canvas.** If you need to draw an image into a `<canvas>` and export it via `toBlob()`, embed the image as a base64 data URI. Workflow: write template HTML with `__LOGO_B64__` placeholder, generate base64 via `base64 -w 0 logo.png`, substitute in via Python or sed.
2. **Audio autoplay is gated everywhere.** Browsers require a user gesture before audio plays. Use a "click anywhere to start" pill that disappears on first click.
3. **`html2canvas` and Inter font.** If using `html2canvas` for HTML→PNG export (for Play Store screenshots, etc.), wait for `document.fonts.ready` before capture or text will render with fallback fonts.
4. **The timeline must stop looping during recording.** Pattern: `if (loop && !recording) next = next % duration; else { next = duration; setPlaying(false); }`. Otherwise the .webm tail captures Scene 1.
5. **`playbackSpeed` affects timeline only, not audio.** Auto-stop timer uses `duration / playbackSpeed + 0.2` seconds. If VO is wired and timeline is 1.3×, audio desyncs. Revert to 1.0× when adding VO.
6. **Don't put images inside elements you'll `html2canvas`.** Either embed as base64 OR set `useCORS: true` on html2canvas AND serve images with `Access-Control-Allow-Origin: *`. On `file://` neither approach works reliably for cross-folder images — just base64-embed.

### Deliverable file structure

```
[PROJECT_NAME]/
  [PROJECT_NAME].html              ← the demo, fully self-contained
  voiceover.mp3                    ← optional, user-provided
  sounds/                          ← optional, user-provided
    tap.mp3
    whoosh.mp3
    stinger.mp3
```

If the demo needs the logo embedded (for a recording / export), keep a separate `[PROJECT_NAME]-template.html` with `__LOGO_B64__` placeholder and a one-line substitute script.

### How to start

Before writing any code, ask me four clarifying questions:
1. **Audience** — consumer homepage / B2B sales deck / investor pitch
2. **Length** — 30s / 60s / 90s
3. **Audio approach** — voiceover, sound cues only, or both
4. **Theme** — white background or dark background

Then sketch the scene breakdown as a numbered list (one phrase per scene). Confirm with me before writing code.

Default to **skeptical, not supportive**. Before agreeing with anything I say, stress-test it. Tell me what the weakest part of my brief is before you start building.
