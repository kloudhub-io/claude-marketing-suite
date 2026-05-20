---
name: create-onboarding
description: Use when the user wants to generate onboarding slides for an app — typically 5 static images shown to new users right after sign-up. Triggers on "/create-onboarding", "create onboarding slides", "generate onboarding images", "make onboarding screens", "replace existing onboarding". Builds an HTML tool that renders 5 slides at 864×1821 PNG each with title + subtitle baked in, each individually downloadable. Output files match common onboarding asset filenames so they slot into existing React Native / Expo apps. Requires marketing-brief.md.
allowed-tools: Read, Glob, Grep, Write, Bash
---

# Create Onboarding Slides

Build an HTML tool that renders 5 onboarding slides as downloadable PNGs.

## Prerequisites

Read `marketing-brief.md`. If missing, direct user to `/setup-marketing-suite`.

Also: check the codebase for an existing onboarding flow (typically `src/features/onboarding/` or `mobile/src/onboarding/`). If found, note:
- Current slide count
- Existing PNG dimensions
- Filename pattern (e.g., `Slide1.png` ... `Slide5.png`)
- Current copy / illustrations (so you can improve, not replace blindly)

## Spec

- **5 slides** (standard count for onboarding carousels — covers problem → solution → protection → use cases → CTA)
- **864 × 1821 PNG each** — close to flagship phone aspect (~9:18.97), works with `contentFit: contain` for letterboxing
- **Title + subtitle baked into the bottom third** of each slide (top 60% is the illustration / product mockup)
- **Background gradient:** light cream `#F5E8E1` at top transitioning to white `#FFFFFF` at ~70% height (so letterboxing on different phones blends invisibly with the parent screen's matching gradient)
- **Each downloadable individually** via a Save button per slide
- **Filenames:** `Slide1.png` through `Slide5.png` — drop directly into the existing app's onboarding asset folder

## Standard slide arc (adapt copy to product)

<slide_arc>
**Slide 1 — Problem** (what's wrong with the current state)
- Title: e.g., "Some things shouldn't live forever."
- Visual: A small, deliberate composition representing the problem (stacked old documents, a chat thread that won't go away, etc.)

**Slide 2 — Solution mechanism** (the key feature)
- Title: e.g., "Disappears on your terms."
- Visual: The hero product UI element (a timer, a privacy toggle, the main control)

**Slide 3 — Protection / control** (what stops abuse)
- Title: e.g., "Locked the moment it opens."
- Visual: The product UI in its protective state — a phone with security pills, a lock overlay, etc.

**Slide 4 — Use cases** (variety of what they can do)
- Title: e.g., "Share anything."
- Visual: A 2×2 grid of realistic content previews (not abstract iconography — see anti-AI rules)

**Slide 5 — CTA** (ready to start)
- Title: e.g., "Ready when you are."
- Visual: Just the logo, large, centered. No supporting decoration.
</slide_arc>

## Tech setup

Pure HTML/CSS + html2canvas (no React/Babel — onboarding slides are static, simpler stack).

```html
<script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
```

Each slide is a 864×1821 div with:
- The gradient background baked in
- Visual area (upper 60%)
- Text area (lower 30%)
- Padding at the bottom (~280px) reserved for the in-app dots + Next button overlay so they don't collide with the slide's own text

Single HTML page shows all 5 slides scaled down for preview, each with a Save button.

## Logo embedding

If any slide uses the product logo (Slide 5 typically does), embed it as base64 in the HTML to avoid `file://` canvas-taint when exporting via html2canvas. Pattern:

1. Write the HTML with `__LOGO_B64__` placeholder
2. Generate base64: `base64 -w 0 assets/logo.png > /tmp/logo.b64`
3. Substitute via Python:
   ```python
   tpl = open('onboarding-slides-template.html').read()
   b64 = open('/tmp/logo.b64').read().strip()
   open('onboarding-slides.html','w').write(tpl.replace('__LOGO_B64__', b64))
   ```

## Anti-AI rules — critical for onboarding

Onboarding is the first thing a user sees. AI-templated onboarding feels generic and erodes trust. Strict rules:

1. **No emojis anywhere**
2. **No decorative glows** unless they serve focus on a single focal element
3. **No concentric rings** around icons
4. **No random rotations** — deliberate stack angles only (e.g., `-3°, 2°, -1°` not `-12°, 8°, -6°, 10°, -2°`)
5. **No abstract icons-as-content** — Slide 4 in particular must use realistic doc/content previews, not iconic representations like a giant `$` or monospace digits
6. **No em-dashes in body copy** — use commas
7. **One strong focal element per slide** — Slide 5 in particular: JUST the logo. Not "logo + send pill + 10-sec chip + glow."

## Workflow

1. Read `marketing-brief.md` and check the existing onboarding folder if one exists.
2. Propose the 5 titles + subtitles, customized to the product. Get user confirmation.
3. Build the HTML with the 5 slides + Save buttons.
4. Show user. Ask them to save Slide1.png as a first check.
5. Iterate on the visual treatment of each slide based on feedback. Slides 1, 4, and 5 typically need the most iteration (they're the ones most prone to AI-template looks).

## Output

```
[product-name]-marketing-suite/
  onboarding-slides.html              ← the tool (open in browser)
  onboarding-slides-template.html     ← base64-placeholder source (delete after final)
```

User downloads `Slide1.png` through `Slide5.png` via the in-page Save buttons.
