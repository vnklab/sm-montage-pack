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
```typescript
const BG   = "#000000"
const W    = "#FFFFFF"
const BLUE = "#0A84FF"   // iOS system blue — primary accent
const SUB  = "rgba(255,255,255,0.68)"   // secondary text
const DIM  = "rgba(255,255,255,0.36)"   // tertiary / labels
// No purple. Only BLUE + WHITE accents.
```

### Typography
| Role | Font | Size range | Weight |
|------|------|-----------|--------|
| Hero / display | **Widock Bold** (`TW`) | 68–200 px | 700 |
| Secondary / body | **Manrope** (`MN`) | 20–34 px | 400–800 |
| Terminal / code UI | Menlo, Monaco (`MONO`) | 18–22 px | 400 |

**Hierarchy rule**: Widock for big headlines → Manrope for body/labels → MONO for code/progress only.
**Never** mix Widock and Manrope at the same visual weight on the same line.
**Vertical rhythm**: `lineHeight: 1.4–1.65` for Manrope body. Gap between type blocks: `gap: 10–32px` in flex column.

### Spring configs
```typescript
const BOUNCE = { damping: 16, mass: 1.0, stiffness: 120 }  // iOS entry, slight overshoot
const SOFT   = { damping: 22, mass: 1.0, stiffness: 90 }   // panels, photos — smooth settle

// Helper wrappers:
const fi = (f, f0, f1, v0, v1) =>
  interpolate(f, [f0, f1], [v0, v1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });

const sp = (f, delay, cfg = BOUNCE) =>
  spring({ frame: Math.max(0, f - delay), fps: 30, config: cfg });
```

### Grid — VpnReel style (real 1px lines)
```typescript
const Grid: React.FC<{ opacity?: number }> = ({ opacity = 1 }) => (
  <div style={{
    position: "absolute", inset: 0, pointerEvents: "none",
    backgroundImage: [
      `linear-gradient(rgba(255,255,255,0.07) 1px, transparent 1px)`,
      `linear-gradient(90deg, rgba(255,255,255,0.07) 1px, transparent 1px)`,
    ].join(","),
    backgroundSize: "88px 88px",
    opacity,
    WebkitMaskImage: "radial-gradient(ellipse 85% 85% at 50% 50%, black 15%, rgba(0,0,0,0.5) 55%, transparent 100%)",
    maskImage:       "radial-gradient(ellipse 85% 85% at 50% 50%, black 15%, rgba(0,0,0,0.5) 55%, transparent 100%)",
  }} />
);
// 0.07 opacity = subtle; increase to 0.11 for more visible grid
```

### Camera drift
```typescript
const Cam: React.FC<{ frame: number; amp?: number; children: React.ReactNode }> = ({ frame, amp = 4, children }) => (
  <div style={{
    position: "absolute", inset: 0,
    transform: `translate(${Math.sin(frame * 0.016) * amp}px, ${Math.cos(frame * 0.011) * amp * 0.5}px)`,
  }}>
    {children}
  </div>
);
// amp=4 for scenes, amp=5 for CTA
```

---

## TELEGRAM UI SYSTEM (VNK Nano Reel pattern)

Use for any reel showing a Telegram bot interface.

### Flat dark palette — NO gradients in TG UI
```typescript
const TG_BG     = "rgba(17, 17, 23, 0.97)"   // phone body
const TG_HEADER = "rgba(24, 24, 32, 1)"        // header bar
const TG_MSG_IN = "rgba(36, 36, 48, 1)"        // incoming bubble
const TG_PANEL  = "rgba(30, 30, 40, 1)"        // settings panel, buttons
const TG_DIM    = "rgba(255,255,255,0.52)"      // secondary text in chat
const TG_SUBDIM = "rgba(255,255,255,0.36)"      // dim/italic annotations
const TG_BORDER = "rgba(255,255,255,0.10)"      // subtle dividers
```

### TgPhone component
```typescript
// width: 660px — not full screen, gives breathing room
// iOS superellipse corners: borderRadius: 34
// Branded blue border + glow — separates window from dark background:
border: "1.5px solid rgba(10,132,255,0.55)"
boxShadow: "0 0 0 1px rgba(10,132,255,0.12), 0 0 60px rgba(10,132,255,0.22), 0 20px 60px rgba(0,0,0,0.80)"
// Header: flat TG_HEADER, avatar 54px, bot name Manrope 700 26px WHITE, subtitle 18px TG_DIM
// NO online status dot
// Entry animation: SOFT spring, scale 0.94→1 + translateY 36→0
```

### Bubbles — flat, no gradients
```typescript
// Outgoing (user, right side): solid BLUE
background: BLUE
borderRadius: "28px 28px 6px 28px"   // sharp corner bottom-right (tail)

// Incoming (bot, left side): flat dark
background: TG_MSG_IN
border: "1px solid rgba(255,255,255,0.08)"
borderRadius: "6px 28px 28px 28px"   // sharp corner top-left (tail)
```

### CRITICAL: Bubble alignment
```typescript
// BubbleOut has alignSelf: "flex-end" — only works when parent is a FLEX container
// Animation wrapper divs MUST have: display: "flex", flexDirection: "column"
// Otherwise alignSelf is ignored and bubbles appear left-aligned

// ✅ Correct:
<div style={{ display: "flex", flexDirection: "column", opacity: ..., transform: ... }}>
  <BubbleOut frame={frame} delay={8}>...</BubbleOut>
</div>

// ❌ Wrong (block div breaks alignSelf):
<div style={{ opacity: ..., transform: ... }}>
  <BubbleOut ...>...</BubbleOut>
</div>
```

### PhotoBubble — full-width 1:1
```typescript
// DO NOT wrap in UserRow — use standalone with flex wrapper
// width: "82%", aspectRatio: "1/1", objectFit: "cover"
// side="out" → alignSelf: "flex-end", borderRadius: "26px 26px 6px 26px"
// side="in"  → alignSelf: "flex-start", borderRadius: "6px 26px 26px 26px"
// Entry: SOFT spring, scale 0.92→1
```

### Buttons — flat, no gradients
```typescript
// Regular button:
background: TG_PANEL
border: "1px solid rgba(255,255,255,0.10)"
borderRadius: 18   // iOS superellipse
padding: "15px 16px"
fontFamily: MN, fontSize: 22, fontWeight: 600

// Accent button (primary action):
background: "rgba(10,132,255,0.28)"
border: "1px solid rgba(10,132,255,0.65)"

// Icon + label button: use justifyContent: "center", gap: 10 — always center text
```

### BotRow / UserRow
```typescript
// BotRow: nano logo 46px avatar + BLUE "VNK Nano 2.0" label Manrope 700 22px
// UserRow: BLUE circle avatar "I" + "rgba(255,255,255,0.75)" name label
// Avatar always 46×46px, borderRadius: "50%"
```

### TgSpinner — BLUE arc (not white)
```typescript
// stroke: BLUE for active spinner (not white)
// size: 34px default, strokeWidth: 2.5
// rot = (frame * 8) % 360
// arc = circ * 0.30
```

### Progress bar — MONO
```typescript
const ProgressBar: React.FC<{ progress: number }> = ({ progress }) => {
  const filled = Math.round(progress * 10);
  return (
    <div style={{ fontFamily: MONO, fontSize: 22, color: W, letterSpacing: 2 }}>
      {"■".repeat(filled)}{"□".repeat(10 - filled)}{"  "}{Math.round(progress * 100)}%
    </div>
  );
};
```

---

## LUCIDE ICONS (use instead of emoji)

Always use Lucide SVG paths — stroke only, `strokeWidth: 1.75`, `color: BLUE`, `viewBox: "0 0 24 24"`.
**Never use emoji as icons in compositions.**

```typescript
// Sparkles — generation / AI magic
<svg width={38} height={38} viewBox="0 0 24 24" fill="none" stroke={BLUE} strokeWidth="1.75" strokeLinecap="round" strokeLinejoin="round">
  <path d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L21 12l-5.813-1.912a2 2 0 0 1-1.275-1.275L12 3Z"/>
  <path d="M5 3v4"/><path d="M19 17v4"/><path d="M3 5h4"/><path d="M17 19h4"/>
</svg>

// PenLine — edit / photo editing
<svg ...>
  <path d="M12 20h9"/>
  <path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/>
</svg>

// MessageSquare — chat / text models
<svg ...>
  <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
</svg>

// Check — success / ready
<svg width={26} height={26} ... strokeWidth="2.2">
  <polyline points="20 6 9 17 4 12"/>
</svg>

// Bot — AI model active badge
<svg ...>
  <path d="M12 8V4H8"/>
  <rect width="16" height="12" x="4" y="8" rx="2"/>
  <path d="M2 14h2"/><path d="M20 14h2"/>
  <path d="M15 13v2"/><path d="M9 13v2"/>
</svg>

// Share2 — network/routing (from VpnReel carousel)
// LayoutGrid — services grid
// EyeOff — privacy
// See Carousel.tsx for full paths
```

---

## BORDER RADIUS — iOS Superellipse system

```
Phone/dialog containers: borderRadius: 34
Feature cards:           borderRadius: 32
Settings panels:         borderRadius: 22
Buttons:                 borderRadius: 18
Bubble tails (sharp):    6px (not 4px)
Bubble body:             28px
Photo bubbles:           26px body, 6px tail
Badge/pill:              borderRadius: 40
Model tag:               borderRadius: 18
Before/After block:      borderRadius: 26
```

---

## BADGE / PILL STYLE (key UI pattern)

```typescript
// Blue pill badge — for labels like "прямо в TELEGRAM", "ЧАТ С текстовыми моделями"
{
  background: "rgba(10,132,255,0.20)",
  border: "1px solid rgba(10,132,255,0.70)",
  borderRadius: 40, padding: "14px 34px",
  fontFamily: MN, fontSize: 28, fontWeight: 800, color: W,
  letterSpacing: "0.06em", textTransform: "uppercase",
  boxShadow: "0 0 32px rgba(10,132,255,0.25)",
}
// Use for: feature labels, section badges, hook identifiers
```

---

## FEATURE CARDS (hook scene pattern)

```typescript
// 3 capability cards, centered, width: 600px
{
  background: "rgba(255,255,255,0.10)",
  border: "1.5px solid rgba(255,255,255,0.36)",
  boxShadow: "inset 0 1px 0 rgba(255,255,255,0.22)",
  borderRadius: 32, padding: "26px 36px",
  display: "flex", alignItems: "center", gap: 24,
  width: 600,
}
// Title: Widock 30px WHITE — NOT Manrope
// Subtitle: Manrope 20px TG_DIM
// Icon: Lucide SVG 38px BLUE (left side)
// Enter animation: translateX from -36px, staggered i*8 frames
// Container: gap: 18, alignItems: "center" (centered, not stretched)
```

---

## SCENE TEMPLATES

### Hook scene — 3 capabilities (3.3s / 100f)
```
Layout (top → bottom, gap: 32):
  1. Blue pill badge "прямо в TELEGRAM" — fade in, fromY -20
  2. Hero: Widock 136px WHITE "NANO" + Widock 60px BLUE "2.0" glow
  3. 3 capability cards (Lucide icon + Widock title + Manrope sub)
     — staggered enter from left, delay 18+i*8f
```

### Telegram UI scene — upload/settings (3.3s / 100f)
```
Layout: TgPhone centered, no outer label
  Inside phone:
    - Settings panel (flat TG_PANEL, borderRadius 22)
    - Format grid buttons (2-col or 3-col)
    - PhotoBubble (flex wrapper, side="out")
    - BotRow response with action buttons
```

### Processing scene (3.5s / 105f)
```
  - BotRow with action buttons (persistent from prev scene)
  - User bubble enters (flex wrapper, BubbleOut)
  - BotRow processing: BLUE label "Обрабатываю..." + TgSpinner + ProgressBar
  - progressVal = fi(frame, 28, 90, 0, 0.92)
  - Labels cycle: "Анализирую..." → "Генерирую..." → "Финальная обработка..."
```

### Result scene (3.3s / 100f)
```
  - User bubble (echo of prompt)
  - BotRow: PhotoBubble side="in" + model label BLUE + Check icon + "Готово!"
  - Below phone: Before/After block
    {
      display: "flex", alignItems: "stretch",
      border: "1.5px solid rgba(255,255,255,0.22)",
      borderRadius: 26, overflow: "hidden",
    }
    Left: "До" Widock 34px WHITE, padding 14px 32px, bg rgba(W,0.08)
    Divider: 1px rgba(W,0.18)
    Right: "После" Widock 34px BLUE, padding 14px 32px, bg rgba(BLUE,0.18)
```

### Generation scene (3.3s / 100f)
```
  Top label: "А ещё" + "генерация." Widock 68px BLUE glow
  TgPhone:
    - BubbleOut: typewriter text prompt (fi chars 0→PROMPT.length over 35f)
    - BotRow: "Генерирую..." + TgSpinner + ProgressBar
```

### Gemini / text model scene (3.8s / 115f)
```
Hero (gap: 20):
  1. Blue pill badge: "ЧАТ С текстовыми моделями" (same style as hook badge)
  2. Widock 92px WHITE "Gemini"

TgPhone subtitle: "Gemini 2.0 Flash":
  - Model tag: BLUE bg, IcnBot, WHITE name + BLUE "· активен"
  - User question: BubbleOut typewriter (flex wrapper!)
  - Bot answer: BotRow + BubbleIn typewriter (whiteSpace: "pre-wrap")
```

### CTA scene (6.3s / 190f)
```
  Manrope 34px SUB — tagline "Генерируй. Редактируй. Общайся."
  Widock 200px BLUE — "NANO" with pulsing glow
    pulse = Math.sin(frame * 0.10) * 0.5 + 0.5
    textShadow: `0 0 ${60 + pulse * 50}px rgba(BLUE, ${0.45 + pulse * 0.20})`
  Widock 68px WHITE — "2.0"
  Widock 46px WHITE — "@vnknanobot"
  Manrope 30px BLUE 600 — "Напиши /start — результат через секунды"
  Manrope 22px DIM — "@vnk_lab_bot → все инструменты VNK LAB" (bottom: 160)
```

---

## VNK NANO REEL — scene map reference (900f / 30s)

| # | Scene | Frames | Duration | Purpose |
|---|-------|--------|----------|---------|
| 1 | SceneHook | 0–100 | 3.3s | 3 capabilities: генерация, редактирование, Gemini |
| 2 | SceneUpload | 100–200 | 3.3s | Настройки бота + загрузка фото |
| 3 | SceneProcess | 200–305 | 3.5s | Запрос + прогресс-бар |
| 4 | SceneResult | 305–405 | 3.3s | Результат + До/После |
| 5 | SceneGenerate | 405–505 | 3.3s | Текстовая генерация |
| 6 | SceneGenerateResult | 505–595 | 3s | Готовое изображение |
| 7 | SceneGemini | 595–710 | 3.8s | Gemini чат typewriter |
| 8 | SceneCTA | 710–900 | 6.3s | NANO 2.0 + @vnknanobot |

---

## REMOTION GOTCHAS (always check)

1. **Hooks in loops are illegal** — never call `useCurrentFrame()` inside `.map()`. Use parent `frame`.
2. **`staticFile()`** for all assets — fonts in `public/fonts/`, images in `public/img/`.
3. **Sequence frame is local** — inside `<Sequence from={60}>`, `useCurrentFrame()` returns 0 at frame 60.
4. **No CSS `animation:` keyframes** — drive everything with `frame` + `spring()` / `interpolate()`.
5. **alignSelf requires flex parent** — animation wrapper divs need `display: "flex", flexDirection: "column"` or `alignSelf` is ignored.
6. **PhotoBubble never inside UserRow** — wrap in plain flex div instead, otherwise sharp corner lands on wrong side.
7. **Spring SOFT for panels/photos** — use BOUNCE only for text/hero entries.

---

## WHEN INVOKED

Ask the user for:
1. **Product / service** — what is being promoted?
2. **Key message** — one sentence the viewer must remember
3. **CTA** — what action? (DM, link, comment keyword)
4. **Assets available** — logos, fonts, brand colors, photos
5. **Tone** — dark tech / minimal / energetic / luxury

Then output the full montage pack using the templates above. Always start with the hook that creates tension (3 capability cards), end with CTA that resolves it (pulsing NANO + handle).
