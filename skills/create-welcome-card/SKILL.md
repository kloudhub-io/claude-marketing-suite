---
name: create-welcome-card
description: Use when the user wants a personal/branded square share image to send when introducing the product to others. Triggers on "/create-welcome-card", "create welcome image", "make a share card", "generate a welcome graphic", "design a personal intro image for [product]". Produces a 1080×1080 square PNG with the product logo, a welcome line, a signature from the sender, and the product tagline. Designed to share well as a chat preview on WhatsApp/iMessage and a post image on LinkedIn/Twitter. Requires marketing-brief.md.
allowed-tools: Read, Write, Bash
---

# Create Welcome Card

Build a 1080×1080 square share image. Designed to be sent in chats, posted on social, or used as a profile/cover where 1:1 aspect is preferred.

## Prerequisites

Read `marketing-brief.md`. If missing, direct user to `/setup-marketing-suite`.

Ask the user one question:
- **Signature line** — "Who's the sender? (e.g., 'from Sarah', '~ Sarah', 'shared by the [Product] team')"

That single answer locks the personalization. Everything else comes from the brief.

## Spec

- **1080 × 1080 PNG**
- Centered composition, vertically stacked:
  1. **Logo** (~260px, centered)
  2. **Welcome headline** (e.g., "Welcome to [Product].") — 80–96px, 800-weight, dark text
  3. **Signature line** (e.g., "~ from Sarah") — 32–40px, brand-color accent, lighter weight
  4. **Tagline** (from `marketing-brief.md`) — 28px, muted-text color, distance from signature
- White background with subtle dot grid (5% opacity, 40px spacing)
- Single accent color: brand primary

## Tech setup

Pure HTML/CSS + html2canvas. Logo embedded as base64 (avoid `file://` canvas-taint).

```html
<div class="welcome">
  <img class="logo" src="data:image/png;base64,__LOGO_B64__" alt="[Product]"/>
  <h1>Welcome to [Product].</h1>
  <div class="from">[signature]</div>
  <div class="tag">[tagline from brief]</div>
</div>
```

Save button uses `html2canvas → toBlob → <a download>` pattern with output filename `[product]-welcome-from-[name].png`.

## Build pattern

1. Write template HTML (`welcome-card-template.html`) with `__LOGO_B64__` placeholder
2. Generate base64: `base64 -w 0 assets/logo.png > /tmp/logo.b64`
3. Substitute via Python:
   ```python
   tpl = open('welcome-card-template.html').read()
   b64 = open('/tmp/logo.b64').read().strip()
   open('welcome-card.html','w').write(tpl.replace('__LOGO_B64__', b64))
   ```

## Anti-AI rules

This is a personal-feel asset, so AI-template patterns hurt extra here. Strict:

1. **No emojis** in the headline, signature, or tagline
2. **No em-dashes** in body copy — if the user wants a separator before the signature, suggest `~` (tilde) or `·` (middle dot) instead
3. **No glow behind the logo** — the bare logo on white reads as more personal than a logo with a halo
4. **One accent color** — only the brand primary. Don't introduce a second.

## Workflow

1. Read `marketing-brief.md`. Ask for the signature line.
2. Build the HTML.
3. Show user. Iterate on signature placement/weight if needed.
4. Tell user to click Save to download.

## Output

```
[product-name]-marketing-suite/
  welcome-card.html             ← the tool
  welcome-card-template.html    ← base64-placeholder source (safe to delete after)
```

Downloaded PNG: `[product]-welcome-from-[name]-1080x1080.png`.
