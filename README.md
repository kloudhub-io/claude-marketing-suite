# Marketing Suite — Cowork Plugin

Generate a complete marketing asset suite — feature demo video, vision film, onboarding slides, Play Store screenshots, feature graphic, and welcome card — from any product's frontend codebase.

## What this gives you

Six slash commands, each producing a deliverable that lives in a `[product-name]-marketing-suite/` folder:

| Command | Produces |
|---|---|
| `/setup-marketing-suite` | Reads codebase, extracts design tokens, writes `marketing-brief.md`, asks the 4 questions that lock the spec |
| `/create-feature-demo` | 60s animated product demo with recording (`feature-demo.html`) |
| `/create-vision-film` | 20s cinematic brand film (`vision-film.html`) |
| `/create-onboarding` | 5 onboarding slides at 864×1821 (`onboarding-slides.html`) |
| `/create-store-assets` | Feature graphic (1024×500) + phone screenshots (1080×1920 × 4) + 7-inch tablet (2560×1440 × 4) + 10-inch tablet (3200×1800 × 4) |
| `/create-welcome-card` | 1080×1080 personal share image (`welcome-card.html`) |

## Workflow

1. Mount or open your product's frontend codebase in Cowork
2. Run `/setup-marketing-suite` — this is the recon phase, has to happen first
3. Answer the 4 clarifying questions when asked
4. Run any of the `create-*` commands as needed (in any order, but `create-feature-demo` is the best first test)
5. Each asset opens in your browser; click Save / Record to download

## Installation

This repo is published as a Claude plugin marketplace. Installation is a two-step flow: first you **add the marketplace** (the GitHub repo), then you **install the plugin** from that marketplace.

### Option A — Install via Claude Code / Cowork CLI (recommended)

Run these two commands inside Claude Code or Cowork:

```
/plugin marketplace add https://github.com/kloudhub-io/claude-marketing-suite
/plugin install marketing-suite@claude-marketing-suite
```

That's it. The six slash commands become available immediately:

- `/setup-marketing-suite`
- `/create-feature-demo`
- `/create-vision-film`
- `/create-onboarding`
- `/create-store-assets`
- `/create-welcome-card`

### Option B — Install via Cowork Settings UI

1. Open Cowork → **Settings → Plugins**
2. Click **Add plugin source** (or **Add marketplace**)
3. Paste: `https://github.com/kloudhub-io/claude-marketing-suite`
4. Once the marketplace is added, install the `marketing-suite` plugin from it

### Option C — Install from .plugin file (offline / one-off)

1. Download `marketing-suite.plugin` from this repo
2. Open it in Cowork
3. Click Install

### Updating to a new version

When this repo is updated, run:

```
/plugin marketplace update claude-marketing-suite
/plugin update marketing-suite
```

### Uninstall

```
/plugin uninstall marketing-suite
/plugin marketplace remove claude-marketing-suite
```

## Architecture

```
claude-marketing-suite/
├── .claude-plugin/
│   ├── marketplace.json                       ← marketplace manifest (for /plugin marketplace add)
│   └── plugin.json                            ← plugin manifest
├── skills/
│   ├── setup-marketing-suite/SKILL.md         ← run this first
│   ├── create-feature-demo/
│   │   ├── SKILL.md                           ← 60s product demo
│   │   └── references/
│   │       ├── architecture.md                ← Stage/Sprite/Camera primitives
│   │       ├── recording.md                   ← .webm export via getDisplayMedia
│   │       └── audio.md                       ← VO vs sound cues
│   ├── create-vision-film/
│   │   ├── SKILL.md                           ← 20s brand film
│   │   └── references/
│   │       └── particle-dissolve.md           ← the vanish effect
│   ├── create-onboarding/SKILL.md             ← 5 onboarding slides
│   ├── create-store-assets/SKILL.md           ← Play Store graphics
│   └── create-welcome-card/SKILL.md           ← share image
├── docs/                                      ← supplementary prompts (no install needed)
├── marketing-suite.plugin                     ← packaged plugin file
├── LICENSE                                    ← MIT
└── README.md
```

## What works well

- **Architecture is product-agnostic** — Stage/Sprite/Camera/TimelineContext composition has been tested across multiple product types
- **Recording flow is robust** — getDisplayMedia + MediaRecorder + auto-download .webm works on every modern Chrome
- **Design discipline is documented** — nine anti-AI-template rules baked into each skill
- **Static assets export cleanly** — html2canvas at native dimensions, no scaling artifacts

## What needs human judgment

- **Brand voice and copy.** Claude proposes; you correct. The first-take taglines are usually 70% right.
- **"Pixel perfect" is aspirational.** Realistic expectation: 80-90% visual match to your real app. Depends on how clean your design tokens are.
- **Scene ordering for the demo.** Claude proposes a structure; push back if a beat feels weak.
- **Sound design.** Claude wires the cues; you supply the `.mp3` files.

## What won't work

- Products without a frontend codebase (Claude needs something to read)
- Products with very abstract / non-visual UIs (CLI tools, API services)
- Apps where the demo needs real generated user data (e.g., fitness trackers, analytics dashboards) — Claude can mock screens but can't fake compelling data

## Provenance

Extracted from building the [Fliko](https://fliko.in) marketing suite — feature demo, vision film, pitch video, Play Store assets, onboarding slides. Patterns tested on a real production app.

## License

MIT
