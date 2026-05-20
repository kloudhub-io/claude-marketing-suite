---
name: setup-marketing-suite
description: Use when the user wants to begin generating marketing assets for a product they have a frontend codebase for. Triggers on "/setup-marketing-suite", "set up marketing suite for [product]", "start marketing assets", "analyze [product] for marketing", "scan my product for a demo video", "kickstart marketing assets". This is the first step before any create-* skill — it reads the codebase, extracts design tokens (colors, fonts, logo, screen layouts), produces a written brief, and asks the four clarifying questions that lock the spec for downstream asset generation.
allowed-tools: Read, Glob, Grep, Write, Bash
---

# Setup Marketing Suite

Run this first, before any `create-*` skill. The output of this skill is a `marketing-brief.md` file plus the user's answers to four clarifying questions. Every downstream skill reads from this brief.

## What you produce

A file at `[product-marketing-suite]/marketing-brief.md` containing:

1. Product identity (name, one-liner, primary value prop)
2. Design tokens extracted from the codebase
3. Inventory of primary product screens
4. Audience and tone (from user answers)
5. Audio approach for video assets (from user answers)

Plus a folder structure ready for downstream skills to write into.

## Step 1: Reconnaissance — read the codebase

Do this before asking the user anything. Use the tools you have (Glob, Grep, Read) to extract:

<extraction_targets>
**Design tokens** — look in these typical locations:
- `theme.ts`, `colors.ts`, `tokens.ts`, `palette.ts`
- `src/theme/`, `src/design-system/`, `src/styles/`
- `tailwind.config.js`, `tailwind.config.ts`
- `app/styles/`, `mobile/src/theme/`
Record exact hex values for: primary brand color, ink/foreground text, muted text, background, surface, accent, success, danger.

**Typography** — find the font family:
- Look in `index.html`, `_document.tsx`, theme files, or font imports
- Note family + key weights used (usually 400, 500, 600, 700, 800)
- Default to Inter if not specified

**Logo / app icon** — search for:
- `assets/icons/`, `app-icon/`, `public/`, `static/`
- Files matching `*logo*`, `*icon*`, `foreground.png`, app-icon adaptive-icon directories, any folder with `*-icon*` in the name
- Record the path to the highest-resolution version

**Primary screens** — find the user-facing screen components:
- React Native: look in `src/features/*/`, `src/screens/`, `mobile/src/`
- Web: look in `app/`, `pages/`, `src/routes/`, `src/views/`
- For each screen, note: file path, what the user sees, key elements (header, primary CTA, file/data display, settings)

**Onboarding flow** (if present) — look in `src/features/onboarding/`, `src/onboarding/`, similar paths. Record current slide count and content.

**Product positioning** — read in this order:
- `README.md` at repo root
- `package.json` description field
- `app.json` `name` and `slug` (React Native)
- Any marketing copy in `docs/`, `marketing/`, pitch docs
</extraction_targets>

If you cannot find design tokens or the logo, do not invent them. Stop and ask the user where to find them.

## Step 2: Produce the brief

Write the extracted findings to `marketing-brief.md` at the chosen output location. Format:

<brief_format>
```markdown
# Marketing brief — [product-name]

## Identity
- **Name:** [product-name]
- **One-liner:** [from README or inference, flagged if inferred]
- **Tagline (proposed):** [your suggestion, to be confirmed]

## Design tokens
- Primary brand color: #RRGGBB
- Ink (text): #RRGGBB
- Muted text: #RRGGBB
- Background: #RRGGBB
- Surface: #RRGGBB
- Accent: #RRGGBB
- Semantic — Success: #RRGGBB, Danger: #RRGGBB

## Typography
- Family: [name]
- Weights used: [400, 500, 600, 700, 800]

## Logo
- Source path: [path/to/logo.png]
- Copied to: assets/logo.png

## Primary screens identified
| Screen | File | Purpose |
|---|---|---|
| HomeScreen | src/... | Entry point, primary CTA |
| ... | ... | ... |

## Onboarding flow (if found)
[Slide count + content per slide]

## Audience guess
[Consumer / B2B / Prosumer], based on [reasoning].

## Pending — to be locked by user answers
- Audience confirmation
- Tagline confirmation
- Audio approach for demo (VO vs sound cues)
- Anything to NOT show in mockups
```
</brief_format>

## Step 3: Folder scaffolding

Create the output folder structure:

```bash
mkdir -p [product-name]-marketing-suite/assets
```

Copy the logo from the codebase to `[product-name]-marketing-suite/assets/logo.png`. Then generate the base64 version for asset HTMLs that need it embedded:

```bash
base64 -w 0 [product-name]-marketing-suite/assets/logo.png > [product-name]-marketing-suite/assets/logo.b64.txt
```

This base64 file is what the static-asset skills (`create-welcome-card`, `create-onboarding`, etc.) embed into their HTML to avoid `file://` canvas-taint issues.

## Step 4: Ask the four clarifying questions

After producing the brief, ask exactly these four questions using the AskUserQuestion tool (or equivalent):

<clarifying_questions>
1. **Audience** — Who's the asset for?
   - Consumer (homepage opener, app store, social)
   - B2B / Prosumer (sales decks, internal demos)
   - Investor / Press (pitch contexts)

2. **Tagline** — Confirm or correct: "[your proposed tagline]"

3. **Audio for the demo video** —
   - Voiceover (you provide the .mp3)
   - Sound cues only (discrete clicks, whooshes, stingers)
   - Both

4. **Anything sensitive to NOT show** — internal-only screens, real customer data, half-built features?
</clarifying_questions>

Wait for the user's answers. Update `marketing-brief.md` with the locked values. Now downstream skills can run.

## Step 5: Confirm and hand off

Tell the user:
- Brief written to `[product-name]-marketing-suite/marketing-brief.md`
- They can now run any of: `/create-feature-demo`, `/create-vision-film`, `/create-onboarding`, `/create-store-assets`, `/create-welcome-card`
- Recommend starting with `/create-feature-demo` since it's the most informative test of the design-token extraction

## What to NOT do in this skill

- Do not invent design tokens. If the codebase doesn't expose them clearly, ask.
- Do not write any HTML assets yet — that's the job of the create-* skills.
- Do not propose the scene breakdown or copy yet — that happens in the per-asset skill.
- Do not assume the user wants every asset. They may only need 2 of the 6.
