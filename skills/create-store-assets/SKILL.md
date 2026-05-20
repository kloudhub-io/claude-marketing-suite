---
name: create-store-assets
description: Use when the user wants to generate Play Store / App Store listing assets — feature graphic and phone+tablet screenshots. Triggers on "/create-store-assets", "create play store screenshots", "make app store images", "generate listing assets", "build play store feature graphic", "create tablet screenshots". Produces a feature graphic (1024×500), 4 phone screenshots (1080×1920), 4 × 7-inch tablet screenshots (2560×1440), and 4 × 10-inch tablet screenshots (3200×1800). All as downloadable PNGs via html2canvas. Requires marketing-brief.md.
allowed-tools: Read, Glob, Grep, Write, Bash
---

# Create Store Assets

Build the full set of Play Store listing graphics:

| Asset | Dimensions | Count | Aspect |
|---|---|---|---|
| Feature graphic | 1024 × 500 | 1 | 16:5 (Play Store fixed spec) |
| Phone screenshots | 1080 × 1920 | 4 | 9:16 portrait |
| 7-inch tablet screenshots | 2560 × 1440 | 4 | 16:9 landscape |
| 10-inch tablet screenshots | 3200 × 1800 | 4 | 16:9 landscape |

All deliverables: PNG, downloadable individually via html2canvas Save buttons.

## Prerequisites

Read `marketing-brief.md`. If missing, direct user to `/setup-marketing-suite`.

## Files to produce

```
[product-name]-marketing-suite/
  play-store-feature-graphic.html       ← 1024×500 feature graphic tool
  play-store-screenshots.html           ← 4 phone screenshots
  play-store-tablet-screenshots.html    ← 4 × 7-inch + 4 × 10-inch
```

Build them in this order: feature graphic first (simplest), then phone screenshots (most visual work), then tablet screenshots (derived from phone screenshots).

## Feature graphic (1024 × 500)

Layout:
- **Left 35%** — product logo (large, ~240px) + wordmark below
- **Right 65%** — eyebrow (brand-color uppercase, small) + big tagline (~80px, 800-weight, two lines) + subline (~22px, muted color, optional second line)
- Background: white with a very subtle dot grid (5% opacity)
- Single accent color: the brand's primary

The big tagline is the message you want the listing page to scream. Examples:
- "Files that vanish."
- "Inbox zero. Daily."
- "Money, finally simple."

Subline gives the supporting one-liner.

## Phone screenshots (1080 × 1920, ×4)

Each screenshot is a marketing graphic, not a bare app screenshot. Composition:

- **Top 25%** — Eyebrow + big headline introducing the scene's value prop
- **Middle 60%** — Phone frame containing the actual app UI for that scene
- **Bottom 15%** — Supporting one-liner

The 4 scenes — adapt to product, but the standard arc:

<phone_screenshots>
1. **Hero / "Send anything privately"** — show the main screen (HomeScreen / Inbox / Dashboard)
2. **Set terms / Configure** — show the main configuration UI (the timer, the privacy toggle, the settings)
3. **Protection / Locked** — show the product in its protective state, with a decorative red badge overlapping (e.g., "Screenshot blocked") if relevant
4. **In control / Audit** — show the audit / history / control panel
</phone_screenshots>

Use the phone UI components built for the feature demo (or rebuild with the same design tokens). The phone is the focal middle 60% of the canvas.

## Tablet screenshots (2560×1440 + 3200×1800)

**Landscape 16:9.** Tablets are typically held landscape, and a landscape layout lets the marketing copy live on the left and the phone UI on the right (instead of cramming text above/below as in the portrait phone screenshots).

Composition:
- **Left 55%** — Eyebrow + big headline + supporting line + feature bullets (3 lines with checkmarks)
- **Right 45%** — Phone frame (scaled to fit), centered vertically

The 4 scenes are the same as phone screenshots — same content, different layout.

**Phone scaling:**
- 7-inch (2560×1440): scale phone to ~82% via CSS zoom (the 720×1480 native phone is too tall for 1440 canvas)
- 10-inch (3200×1800): scale phone to ~110% (room to breathe in the bigger canvas)

## Tech setup

Pure HTML/CSS + html2canvas. JS-driven generation: define screen markup once as a template string, define a `SCENES` config array, generate cards from `SCENES × TABLETS`.

```js
const SCREENS = {
  home: `[markup for HomeScreen]`,
  configure: `[markup for ConfigScreen]`,
  protected: `[markup for ProtectedView]`,
  audit: `[markup for AuditScreen]`,
};

const SCENES = [
  { id: 'hero', screen: 'home', eyebrow: '...', headline: '...', features: [...] },
  // ...
];

const TABLETS = [
  { size: 7,  cls: 'ss-7',  width: 2560, height: 1440 },
  { size: 10, cls: 'ss-10', width: 3200, height: 1800 },
];

// Generate 8 cards by iterating TABLETS × SCENES
```

This pattern keeps the code DRY and lets the user edit content in one place.

## Critical implementation details

1. **CSS scale for previews, native dimensions for export.** Each card's preview shows the screenshot scaled-down (so it fits in the layout), but the underlying `.ss` div stays at native 1080×1920 / 2560×1440 / 3200×1800. html2canvas captures at native dimensions.

2. **`await document.fonts.ready` before capture.** Otherwise Inter falls back to system fonts and text rendering looks different from preview.

3. **Logo as base64 if any screenshot uses it.** Same base64-template-substitute pattern as elsewhere (see `create-onboarding/SKILL.md`).

4. **Phone UI must match the product.** Pull dimensions, colors, fonts from `marketing-brief.md`. The HomeScreen mockup should look like a screenshot of the real app, not an approximation.

## Anti-AI rules

Same as elsewhere:

1. No emojis
2. No decorative glows beyond focus indicators
3. No abstract icons-as-content (real document mockups, not stylized representations)
4. No em-dashes in copy
5. One focal element per screenshot (the phone UI is the focal point — keep marketing copy supporting, not competing)

## Workflow

1. Read `marketing-brief.md`. Extract the design tokens and primary screen list.
2. Propose 4 screenshot themes specific to the product. Get user confirmation.
3. Build feature graphic first (smallest scope). Show user.
4. Build phone screenshots. Show user; iterate on individual scenes as needed.
5. Build tablet screenshots from the same content. Should require minimal new work since the content is shared.
6. Recommend the user download one of each (feature graphic, one phone, one 7-inch, one 10-inch) to verify dimensions are exact.

## Output

```
[product-name]-marketing-suite/
  play-store-feature-graphic.html       ← 1024×500 feature graphic
  play-store-screenshots.html           ← 4 phone screenshots
  play-store-tablet-screenshots.html    ← 4 × 7-inch + 4 × 10-inch
```

Downloaded PNG filenames match Play Console upload requirements directly.
