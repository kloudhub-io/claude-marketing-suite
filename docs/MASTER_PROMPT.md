# Master prompt: build a complete marketing asset suite for any product

Paste the **PROMPT BLOCK** below into a fresh Claude conversation. Replace nothing — the prompt is product-agnostic. Give Claude access to your product's frontend codebase (read-only is enough) and let it ask its own clarifying questions.

What this produces:
- `index.html` — landing page with tabs linking to every asset
- `feature-demo.html` — animated 60s product demo (recordable to .webm)
- `vision-film.html` — cinematic 20s brand film
- `onboarding-slides.html` — 5 onboarding images (864×1821, downloadable)
- `play-store-screenshots.html` — 4 × phone + 4 × 7-inch tablet + 4 × 10-inch tablet
- `play-store-feature-graphic.html` — 1024×500 Play Store graphic
- `welcome-card.html` — 1080×1080 share card

---

## THE PROMPT BLOCK

You are building a complete marketing asset suite for the product whose codebase I'm about to give you access to. The deliverable is a folder of self-contained HTML files, plus an `index.html` that ties them together with a tabbed navigation.

### Step 0 — Codebase reconnaissance (do this first, before anything else)

Read the codebase and extract:

1. **Design tokens** — look for `theme.ts`, `colors.ts`, `tokens.ts`, `design-system/`, `tailwind.config`, or similar. Record exact hex values for: primary brand color, ink/text color, muted text color, background, surface, accent, semantic colors (success/danger).
2. **Fonts** — find what font family the app uses. If a Google Font, note the family + weights. If a system font, default to Inter.
3. **Logo** — find the app icon / logo asset. Look in `assets/`, `public/`, `icons/`, `app-icon/`. Copy the foreground PNG to the marketing suite's `assets/` folder.
4. **Screens / pages** — list every primary screen the user sees: Home, Create, Detail, Settings, Profile, etc. For each, identify the component file and read its JSX/TSX.
5. **Navigation structure** — what's the user's primary flow? Onboarding → Home → Action → Result?
6. **Product positioning** — read the README, package.json description, app.json display name, any marketing copy in the repo, any pitch deck or vision doc.

Output your findings as a brief at the start of the conversation:

```
Product: [name]
One-liner: [from README or your inference]
Primary brand color: #RRGGBB
Ink: #RRGGBB
Background: #RRGGBB
Font: [family + key weights]
Logo: [path you copied from]
Primary screens identified:
  - HomeScreen → src/...
  - ...
Onboarding flow: [if found]
Audience guess: [consumer / B2B / prosumer]
```

**Then ask me four clarifying questions before writing any code:**
1. Confirm the audience and use-case I should aim at
2. What's the one-sentence value proposition / tagline?
3. Voiceover or sound cues only for the demo video?
4. Anything sensitive I should *not* show in mockups (real customer data, internal-only screens)?

Wait for my answers before building.

### Step 1 — Folder structure

Create the folder `[product-name]-marketing-suite/` at the chosen output location, with:

```
[product-name]-marketing-suite/
  index.html                       ← tabbed landing page
  feature-demo.html                ← 60s product demo
  vision-film.html                 ← 20s vision film
  onboarding-slides.html           ← 5 onboarding images
  play-store-feature-graphic.html  ← 1024×500
  play-store-screenshots.html      ← phone 1080×1920 × 4
  play-store-tablet-screenshots.html  ← 7-inch + 10-inch × 4 each
  welcome-card.html                ← 1080×1080 share image
  assets/
    logo.png                       ← copied from product codebase
    logo.b64.txt                   ← base64-encoded logo, embedded into HTML files that need it
  README.md                        ← what's here, how to use each file
```

### Step 2 — `index.html` (the landing page with tabs)

A simple white page with:
- Header: product logo + name + "Marketing Suite"
- Tab nav: **Demo · Vision · Onboarding · Store screenshots · Feature graphic · Welcome card**
- Each tab links to the corresponding HTML file (open in same tab — these are tools, not embedded views)
- Below the tabs, a brief description of each asset and its intended use

Make `index.html` visually clean — single accent color (the brand color you extracted), Inter font, lots of whitespace. Not flashy. This is an internal tool.

### Step 3 — Build each asset

For each HTML below, follow the architecture and design discipline in the **CROSS-CUTTING RULES** section. Build them in this order so I can review early:

#### 3.1 — Feature demo video (`feature-demo.html`)

A 60-second animated product demo that plays in the browser and is recordable to .webm.

Architecture: `Stage` (1920×1080) + `Sprite` (timeline windows) + `Camera` (keyframed pan/zoom) + `TimelineContext`. React 18 + Babel-Standalone inlined. Single HTML file.

8–12 scenes covering: hook → problem → product reveal → setup flow → recipient/output → why it works → outcome → close.

Mockup the actual product screens, using the design tokens you extracted. Phone shell (or browser shell for web products) with the real screen layouts inside.

Recording button: `getDisplayMedia({ video: true, audio: true })` → MediaRecorder (vp9,opus) → auto-download .webm. Hide all UI chrome when recording starts.

Audio: either a `<VoiceOver src="voiceover.mp3"/>` component (drift formula: `expected = time / playbackSpeed`) OR a `<SoundCue at={2.5}/>` component for discrete cues. Don't do both. Default to sound cues for consumer products.

#### 3.2 — Vision film (`vision-film.html`)

A 20-second cinematic piece. Different from the demo: more emotional, less product-focused. Problem → realization → answer → brand. Dark backdrop. No playback bar. Autoplay + loop. Big confident type, particle dissolves, no UI mockups.

#### 3.3 — Onboarding slides (`onboarding-slides.html`)

5 slides at 864×1821 each (matches modern phone aspect, lets the existing onboarding flow swap them in). Each slide: title + subtitle baked into bottom third, illustration / product mockup in top 60%. Download button per slide.

Topics (adjust to product):
1. **Problem** — what hurts today
2. **Solution mechanism** — the key feature that fixes it
3. **Protection / control** — what stops abuse, what gives the user control
4. **Use cases** — variety of what they can do
5. **CTA** — ready when you are

#### 3.4 — Play Store screenshots (`play-store-screenshots.html`)

4 portrait phone screenshots at 1080×1920 (9:16). Each shows the product UI in a phone frame, with marketing copy above and supporting text below. Marketing overlay covers approximately top 25% and bottom 15%; phone is the focal middle 60%.

Same 4 themes: Hero, Set/Configure, Protection, Stay in control. Adapt to product specifics.

#### 3.5 — Tablet screenshots (`play-store-tablet-screenshots.html`)

4 × 7-inch (2560×1440 landscape 16:9) + 4 × 10-inch (3200×1800 landscape 16:9). Landscape layout: phone on the right (zoomed to fit), marketing copy + feature bullets on the left. Each downloadable separately.

#### 3.6 — Feature graphic (`play-store-feature-graphic.html`)

1024×500 single image. Logo + wordmark on the left, big tagline on the right, supporting line beneath. White background, brand-color accent.

#### 3.7 — Welcome card (`welcome-card.html`)

1080×1080 square share image. Logo + "Welcome to [Product]" + signature line + brand tagline. Designed to be sent in chats, posted on social.

### Step 4 — README

Write `README.md` at the root explaining:
- What each file is for
- How to download each asset
- How to swap in voiceover / sound files
- How to record the demo / vision films
- Filename conventions for replacing files in the actual product (e.g., `Slide1.png` for onboarding)

---

## CROSS-CUTTING RULES

### Tech stack — non-negotiable

- Single HTML file per asset, no build step
- React 18 + Babel-Standalone via unpkg CDN for animated assets (demo, vision)
- Pure HTML/CSS + html2canvas for static assets (screenshots, graphics)
- Must play / export on `file://` (no localhost required)
- Inter font from Google Fonts (or whatever the product uses)
- Inline all logic in `<script>` tags
- Embed images as base64 data URIs where canvas export is involved (avoids `file://` taint)

### Architecture (animated assets)

- **`Stage`** — fixed 1920×1080 canvas, timeline-controlled. Auto-fits to viewport.
- **`Sprite`** — `<Sprite start={5} end={10}>` only renders during that window
- **`Camera`** — interpolates between `{ t, x, y, zoom, ease }` keyframes
- **`TimelineContext`** — exposes `time`, `playing`, `duration`, `playbackSpeed`, `recording`
- **`PlaybackBar`** — play/pause/scrub/restart/record. Hidden during recording.
- Easing helpers: `easeInCubic`, `easeOutCubic`, `easeInOutCubic`, `easeOutQuart`, `easeOutBack` (hand-rolled, no libs)
- Recording auto-stop: `(duration / playbackSpeed + 0.2) * 1000` ms after start
- `loop && !recording` — never wrap during recording

### Audio rules

- VO sync formula: `expected = time / playbackSpeed`. If `Math.abs(audio.currentTime - expected) > 0.35`, seek.
- Audio autoplay is gated everywhere — show a "click anywhere" pill that disappears on first user gesture
- `getDisplayMedia({ audio: true })` as a literal boolean is what triggers the "Share tab audio" toggle. Don't pass an audio-constraints object.
- VO + 1.3× playbackSpeed = desync. If VO is wired, lock `playbackSpeed={1.0}`.

### Design discipline — anti-AI-template rules

These patterns make designs look AI-generated. Avoid them:

1. No emojis anywhere
2. No decorative glows unless they serve focus
3. No concentric rings around icons
4. No random rotations — every rotation should be deliberate
5. No abstract icons-as-content (e.g., a giant `$` for "statement", monospace digits for "OTP")
6. No em-dashes in body copy (use commas or "is")
7. One strong focal element per scene — not "logo + tagline + pill + chip + glow"
8. Show real product UI wherever possible, not metaphorical illustrations
9. Match the product's existing visual identity (extracted in Step 0) — don't invent a new design language

### Gotchas — read before writing code

1. **`file://` canvas taint** — embed images as base64. Write template HTML with `__LOGO_B64__` placeholder, generate base64 via `base64 -w 0 logo.png`, substitute via Python or sed.
2. **`html2canvas` + Inter font** — `await document.fonts.ready` before capture or text falls back to system fonts.
3. **`audio: true` literal boolean** — `getDisplayMedia({ audio: true })` shows the "Share tab audio" toggle. `audio: false` or an object suppresses it.
4. **Timeline loops** — stop looping during recording, otherwise the .webm tail captures Scene 1.
5. **`zoom` vs `transform: scale`** — `zoom` affects layout (good for shrinking phones in tablet screenshots); `transform: scale` doesn't (good for preview scaling).

---

## HOW TO START

When I run this prompt, do the following:

1. **Read the codebase.** Don't ask me where things are; figure it out by exploring `src/`, `app/`, `mobile/src/`, `components/`, etc.
2. **Produce the brief** as described in Step 0 — design tokens, fonts, logo path, screens, audience guess.
3. **Ask the four clarifying questions** about audience, tagline, audio choice, and what not to show.
4. **Wait for my answers.**
5. **Build assets in order** (3.1 → 3.7), checking in with me after each. Don't build all seven before showing anything — I want to react after each one.
6. **Default to skeptical mode.** Before agreeing with anything I say, find the weakest part. If the brief I give you sounds vague, say so.

If you can't find the design tokens or the logo, *don't invent them*. Stop and ask. Pixel-perfect requires accurate input.

---

## END OF PROMPT BLOCK

---

## Turning this into a Cowork plugin (optional)

If you're going to make demos for many products, package this as a Cowork plugin with slash commands:

- `/setup-marketing-suite` — runs Step 0 (codebase recon + brief + clarifying questions)
- `/create-feature-demo` — runs Step 3.1
- `/create-vision-film` — runs Step 3.2
- `/create-onboarding` — runs Step 3.3
- `/create-store-screenshots` — runs Step 3.4 + 3.5
- `/create-feature-graphic` — runs Step 3.6
- `/create-welcome-card` — runs Step 3.7

To build it: use the **`cowork-plugin-management:create-cowork-plugin`** skill that's already available in this Cowork environment. Walk through with that skill, point it at this master prompt as the source-of-truth, and it'll scaffold the plugin manifest, command definitions, and skill files.

The plugin manifest will reference seven skills, each containing the relevant subsection of this master prompt. Slash commands will trigger the corresponding skill.

### Why bother with the plugin

- One product → just paste the master prompt. Plugin is overkill.
- 3+ products → plugin pays off. Slash commands are faster than re-pasting a 5-page prompt each time.
- Selling this as a service → definitely build the plugin. It's the product.
