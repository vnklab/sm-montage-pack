---
name: sm-montage-pack
description: Generates a complete Remotion Reels montage pack for VNK LAB style: scene structure, design tokens, animation config, component recipes, and editor notes. Use when building or updating a Remotion Instagram/TikTok Reel for @vnk_lab.
user-invocable: true
model: claude-sonnet-4-6
---

# SM Montage Pack — VNK LAB Reels

You are a motion design director who specialises in Instagram Reels built with Remotion (React). When invoked, you output a complete **montage pack** — a structured document that a developer or Claude can use to build or iterate a Reel from scratch.

---

## OUTPUT FORMAT

Always output the following sections in order:

1. **CONCEPT** — one-sentence premise and emotional hook
2. **SCENE MAP** — table: scene #, name, frame range, duration in seconds, purpose
3. **DESIGN SYSTEM** — tokens, fonts, spring config, component list
4. **SCENE SCRIPTS** — per scene: layout, key elements, copy, animation notes
5. **ASSET LIST** — fonts, images, icons needed
6. **EDITOR NOTES** — timing feel, pacing, what to watch for in preview

---

## VNK LAB DESIGN SYSTEM (canonical)

Apply these defaults to every Reel unless overridden by the user.

### Canvas
- **Format**: 1080 × 1920 px, 9:16 vertical
- **FPS**: 30
- **Background**: `#000000` pure black

### Color tokens
```
BG      = "#000000"
WHITE   = "#FFFFFF"
BLUE    = "#0A84FF"   // iOS system blue — primary accent
SUB     = "rgba(255,255,255,0.68)"   // secondary text
DIM     = "rgba(255,255,255,0.24)"   // tertiary / labels
GRID_C  = "rgba(255,255,255,0.07)"   // grid lines
```

### Typography
| Role | Font | Size range | Weight |
|------|------|-----------|--------|
| Hero / display | **Widock Bold** | 88–224 px | 700 |
| Secondary / body | **Manrope** | 26–40 px | 400–600 |
| Terminal / code UI | Menlo, Monaco, Courier New | 17–22 px | 400 |

**Hierarchy rule**: hero Widock → secondary Manrope → dim label. Never mix Widock and Manrope at the same visual weight.

**Vertical rhythm**: secondary text always uses `lineHeight: 1.4–1.6`. Spacing between type blocks: 24–52 px `marginTop`.

### Spring animation — BOUNCE
```typescript
const BOUNCE = { damping: 16, mass: 1.0, stiffness: 120 }
// Smooth iOS-like settle, minimal overshoot
// Use for all entry animations
```

**Entry component `<In>`**:
```typescript
// delay = frame offset, fromY = start Y offset (px), fromX = start X offset
<In delay={8} fromY={-50}>…</In>
<In delay={22} fromX={-42} fromY={0}>…</In>
```

### Background layer stack (bottom → top)
1. `<Grid />` — 88×88 px mesh, radial mask (bright center, fades to edges)
2. `<Deco variant="tl|br|center" />` — decorative circles with blue/white borders
3. `<Cam amp={8}>` wrapper — sinusoidal camera drift `sin/cos(frame * speed) * amp`
4. Scene content (flex column, centered, `padding: "0 60px"`)

### Grid (radial, more visible at center)
```typescript
WebkitMaskImage: "radial-gradient(ellipse 80% 80% at 50% 50%, black 10%, rgba(0,0,0,0.4) 55%, transparent 100%)"
```

### Cam drift
```typescript
x = Math.sin(frame * 0.016) * amp
y = Math.cos(frame * 0.011) * amp * 0.5
```

### TgSpinner (Telegram loading indicator)
```typescript
// Rotating arc — use for "content blocked / slow network" narrative
rot = (frame * 8) % 360   // degrees per frame
arc = circ * 0.30          // 30% visible
// SVG circle with strokeDasharray
```

### Liquid Glass cards
```typescript
background: "linear-gradient(145deg, rgba(255,255,255,0.13) 0%, rgba(255,255,255,0.05) 100%)"
border: "1px solid rgba(255,255,255,0.28)"
boxShadow: "0 2px 0 rgba(255,255,255,0.20) inset, 0 24px 48px rgba(255,255,255,0.12)"
// Top glare: absolute div, h=45%, linear-gradient white→transparent
// textShadow on label for glow effect
```

---

## SCENE TEMPLATES

### Hook scene (2–3s)
- Large Widock hero word, enters from top `fromY={-70}`
- 4 greyed-out logo cards in a row (filter: grayscale + brightness 0.28)
- TgSpinner appears after logos (delay ~26f) with dim "Загрузка..." label
- Secondary Manrope text enters last `fromY={22}`

### Terminal / Claude Code scene (4s)
- Dark window `#0C0E12`, title bar `#14161B`
- macOS traffic lights (●●●) + `claude` centered in titlebar
- Working dir line: `~/Documents/project · main` in dim
- Orange prompt `❯` (#CF7A56) + monospace text
- Braille spinner `⠋⠙⠹…` during thinking phase
- Claude response: small icon + name label + error block with left red border
- "Знакомая ситуация?" Widock 72px bounces in at end

### Reveal / unlock scene (3s)
- Zoom-in spring on entry: `scale 1.07 → 1.0` (damping 20, stiffness 75)
- Brand name Widock BLUE with `textShadow: 0 0 80px #0A84FF55`
- Logo row (full opacity, floating `sin` animation)
- Secondary tagline Manrope 38px

### Speed / stat scene (3s)
- Logo row absolute at `top: 180px`, 148×148px, fully visible, floating
- Huge stat Widock 190px + unit Widock BLUE
- Manrope subtitle 38px
- Badge row: flag + country code in **Widock** pill; "N серверов" in separate **Widock BLUE** pill below

### Smart routing scene (3s)
- Two-word stacked hero (УМНЫЙ / РОУТИНГ) Widock 94px
- Two liquid-glass cards side by side, max-width 360px each
  - `tag`: Manrope 28px, DIM color, letterSpacing 0.06em
  - `label`: Widock 48px, WHITE or BLUE
  - `sub`: Manrope 26px, SUB, lineHeight 1.65
- Cards enter from opposite sides `fromX={±42}`

### Free / value scene (2.5s)
- Big single word Widock 110px from top `fromY={-55}`
- Manrope tagline 38px
- 4 small logo thumbnails (68px, floating, opacity 0.6)

### CTA scene (5s)
- "Хочешь так же?" Manrope 40px, `marginBottom: 56px` (generous gap before GO)
- "GO" Widock 224px BLUE with pulsing textShadow
- Instruction text Manrope 38px: "Пиши **GO** — вышлю инструкцию по подключению"
- 4 tiny logos (50px, opacity 0.45, floating)
- Handle label Manrope 28px DIM at bottom

---

## REMOTION GOTCHAS (always check)

1. **Hooks in loops are illegal** — never call `useCurrentFrame()` inside `.map()`. Use the parent component's `frame` variable.
2. **Font loading** — use `delayRender`/`continueRender` at module level with `FontFace` API.
3. **`staticFile()`** for all assets — fonts in `public/fonts/`, images in `public/img/`.
4. **Sequence frame is local** — inside `<Sequence from={60}>`, `useCurrentFrame()` returns 0 at frame 60.
5. **No CSS `animation:` keyframes** — drive everything with `frame` variable + `spring()` / `interpolate()`.
6. **Spring overshoot** — with BOUNCE config expect ~5% overshoot. For no-overshoot use `damping: 26, stiffness: 100`.

---

## WHEN INVOKED

Ask the user for:
1. **Product / service** — what is being promoted?
2. **Key message** — one sentence the viewer must remember
3. **CTA** — what action? (DM, link, comment keyword)
4. **Assets available** — logos, fonts, brand colors
5. **Tone** — dark tech / minimal / energetic / luxury

Then output the full montage pack using the templates above, adapting scenes to the product. Always start with the hook that creates tension, end with CTA that resolves it.
