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

### Option A — Install from GitHub (recommended for ongoing use)

1. Push this folder to a GitHub repo (e.g., `github.com/koushikmalla1/claude-marketing-suite`)
2. In Cowork: Settings → Plugins → Add plugin source
3. Paste your repo URL
4. The 6 slash commands become available immediately

For Claude Code (CLI):
```
/plugin marketplace add https://github.com/koushikmalla1/claude-marketing-suite
/plugin install marketing-suite
```

### Option B — Install from .plugin file (one-off)

1. Open the included `marketing-suite.plugin` file in Cowork
2. Click Install
3. The 6 slash commands become available immediately

## Architecture

```
marketing-suite/
├── .claude-plugin/
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
