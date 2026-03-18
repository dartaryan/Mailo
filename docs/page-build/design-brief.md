# Mailo Landing Page — Design Brief

## Read First
- Read `content-hebrew.md` in this same folder for the actual page content
- This page will live as a single `index.html` on GitHub Pages

---

## The Concept: "שיחת לילה עם מיילו"

The page is a single continuous scroll that looks and feels like a late-night chat conversation. Someone is sitting down with Mailo at 2am asking "how does this work?" and Mailo walks them through the full process, step by step. As you scroll, new messages animate in like chat bubbles with cinematic polish.

**THE KEY INSIGHT: This is a PRACTICAL GUIDE, not a feature list.** The content walks through the 8 steps of working with Mailo in order — from exporting your emails to having a clean, labeled archive with deep biographical insights. Each step explains what happens, who does what (Mailo vs. the user), and what the output is. The commands are mentioned naturally within steps, NOT listed as a standalone section.

This is NOT a literal WhatsApp clone. It's an elevated, editorial interpretation of a late-night text conversation. Think: if a magazine art director designed a page around the *feeling* of texting at 2am in the dark.

---

## Design Specifications

### Color Palette
- **Background:** Deep dark, not pure black. A dark navy-charcoal gradient that feels warm, like a phone screen in the dark. Something like `#0a0e1a` to `#111827`.
- **User bubbles (questions):** Subtle, muted, dark glass with a faint border. The user is asking, not performing. `rgba(255,255,255,0.05)` with a `rgba(255,255,255,0.1)` border.
- **Mailo bubbles (answers):** Warmer, slightly more present. A very subtle warm tint, maybe `rgba(220,38,38,0.06)` background (the faintest hint of Mailo's red). Or a soft amber glow.
- **Accent color:** Mailo red `#DC2626`, used SPARINGLY. Only for: Mailo's name tag, the typing indicator dots, step numbers, maybe a subtle glow behind key moments. NOT for backgrounds or large areas.
- **Text:** Off-white `#e8e0d8` for body (warm, not cold white). Slightly brighter `#f5f0eb` for headings/important text. Muted `#7a7068` for timestamps.
- **Step numbers:** Each step has a bold number (1-8) that could use the accent color or a soft glow. These are the visual anchors of the page.
- **Special:** A very subtle background effect: slow-drifting gradient, or barely-visible floating shapes (opacity 0.02-0.03), or gentle noise texture. Something that makes the background ALIVE but not distracting.

### Typography
- **Hebrew body:** `Heebo` (Google Fonts), clean, modern, great Hebrew support. Weight 300 for body, 500 for emphasis, 700 for Mailo's name and step titles.
- **Monospace (for commands and paths):** `JetBrains Mono` or `Fira Code`, for /mailo-xxx commands and file paths like `inbox/`. Makes them feel like real terminal elements.
- **Direction:** RTL. `dir="rtl"` on html. Command names and file paths are LTR (`direction: ltr; unicode-bidi: embed`).
- **Size:** 17-18px base. Line height 1.8 for Hebrew.

### Layout
- **Max-width:** 720px, centered. This is a conversation, not a dashboard.
- **No cards. No grids. No sections with headers.** The entire page is a flowing chat thread.
- **Each exchange:** A user question bubble (aligned right in RTL), then a Mailo answer bubble (aligned left in RTL). Gap between exchanges: 48-64px.
- **Timestamps:** Small, muted, between exchanges. Progress from "1:47" to "2:23" as you scroll down.
- **Step titles inside Mailo bubbles:** When Mailo says "שלב 1: ייצוא", that title line should be visually distinct — bolder, maybe with a step number that has a subtle accent treatment. But it's INSIDE the bubble, not a section header.
- **Mailo's avatar:** A small 💌 emoji or a tiny red circle with "M", appears next to Mailo's first message and occasionally after, not on every single one.

---

## Animations — THIS IS THE SOUL OF THE PAGE

All animations use CSS only (no JS libraries). IntersectionObserver triggers `.is-visible` class additions.

### Message Entrance (CRITICAL)
- **User bubbles:** Slide in from RIGHT (RTL reading direction) with fade. `transform: translateX(-30px); opacity: 0` to `translateX(0); opacity: 1`. Duration: 0.5s. Easing: `cubic-bezier(0.16, 1, 0.3, 1)` (smooth overshoot).
- **Mailo bubbles:** Slide from LEFT, opposite direction. `transform: translateX(30px)` to `translateX(0)`.
- **Stagger:** Multi-paragraph Mailo answers stagger by 80-120ms per paragraph.

### Typing Indicator (IMPORTANT)
- Before each Mailo answer: three small dots that bounce sequentially.
- Appears for ~800ms, then fades out as the actual answer fades in.
- Use CSS `animation-delay` on three small circles. IntersectionObserver triggers indicator first, then answer after delay.
- Use for the first 3-4 Mailo answers only. After that, skip it — it would get annoying.

### The Pipeline Chain (appears in Step 1 answer)
- 8 small circles or nodes connected by lines, each labeled below with a step name.
- Labels: ייצוא | קליטה | ניפוי | זיהוי אנשים | בניית ציר זמן | שאלות פערים | תובנות עומק | סדר בג'ימייל
- On scroll-trigger: nodes light up sequentially (right-to-left in RTL), 200ms delay each.
- "Light up" = border goes from muted gray to accent red + subtle pulse/glow.
- Connecting lines animate using `stroke-dasharray` + `stroke-dashoffset` (draw effect).
- This gives the user the full map upfront — they see the whole journey before diving in.

### The Sieve Funnel (SHOWPIECE — appears in Step 3)
- Vertical stack of 5 horizontal bars, each narrower than the previous (passes 1-4 narrow, pass 5 is a differently styled bar at the bottom).
- On scroll-trigger: bars animate in from top to bottom with 200ms stagger.
- Width represents emails remaining after each pass.
- Each bar has: pass number, name label, and a small signal description that appears as the bar animates in.
- Gradient from cool (blue/gray for automated passes) to warm (red for content classification).
- Pass 5 bar could glow or pulse differently since it's the only one using LLM.

### Pull Quote Glow
- The "הסטטיסטיקה מוצאת את הסיגנל..." quote gets a faint red glow behind it (`box-shadow: 0 0 40px rgba(220,38,38,0.15)`).
- On scroll-trigger: glow pulses once, expands then settles.

### Scroll Progress Bar
- Very thin (2px) accent-red bar at the very top of the page, fills as you scroll.

### Background Ambient (SUBTLE)
- Option A: Very slow radial gradient that shifts position over 30-60 seconds.
- Option B: 3-5 floating shapes (envelope outlines or circles) at opacity 0.02-0.03, slow CSS keyframe drift.
- Option C: Subtle noise/grain texture overlay at opacity 0.03.
- Pick whichever looks best. If you notice it consciously, it's too much.

---

## Content Structure (from content-hebrew.md)

```
[Scroll progress bar — thin red line at top]
[Ambient background effect]

HERO AREA:
  💌 icon (large, centered)
  "אני מיילו" (large heading, centered)
  One-line subtitle about turning email into life story
  Scroll hint arrow

CHAT THREAD:

  [1:47] User: "אוקיי מיילו, אני רוצה להתחיל. מה עושים?"
  Mailo: Overview + PIPELINE ANIMATION (the full 8-step map)
  
  [1:50] User: "יאללה, שלב 1?"
  Mailo: Step 1 - Export (Google Takeout, user action)
  
  [1:53] User: "אוקיי, הורדתי. מה עכשיו?"
  Mailo: Step 2 - Ingestion (automatic, user waits)
  
  [1:55] User: "ומה קורה עם כל הספאם?"
  Mailo: Step 3 - The Sieve + FUNNEL ANIMATION + PULL QUOTE
  
  [2:01] User: "אוקיי, אז עכשיו יש לך רק את המיילים האמיתיים. מה הלאה?"
  Mailo: Step 4 - People identification (Mailo works, user confirms)
  
  [2:04] User: "ואז?"
  Mailo: Step 5 - Timeline construction (automatic)
  
  [2:07] User: "ואם חסר לך מידע?"
  Mailo: Step 6 - Gap questions (interactive, user answers)
  
  [2:11] User: "ומתי מגיעות התובנות?"
  Mailo: Step 7 - Deep insights (9 depth levels)
  
  [2:16] User: "ובסוף?"
  Mailo: Step 8 - Gmail cleanup (labels + deletions, user approves)
  
  [2:20] User: "כמה זמן כל זה לוקח?"
  Mailo: Timeline + cost summary
  
  [2:23] User: "נשמע טוב. אני מוכן להתחיל."
  Mailo: Call to action + 3 principles to remember

FOOTER:
  "אני מיילו. הנתונים שלך, המודיעין שלי, הסיפור שלך."
  "v1.0.0 | מרץ 2026"
```

---

## Technical Constraints

- **Single HTML file.** CSS in `<style>`, JS in `<script>`, no external files except Google Fonts.
- **No frameworks.** No React, no Tailwind, no GSAP. Pure HTML + CSS + vanilla JS.
- **No build step.** The file IS the deployable artifact.
- **GitHub Pages compatible.** Static HTML, no server-side anything.
- **Mobile responsive.** Chat layout should work beautifully on phones (chat-style pages are natural on phones). Bubbles go full-width on mobile.
- **Performance.** All animations CSS-only (GPU-accelerated transforms + opacity). JS only for IntersectionObserver and progress bar.
- **Accessibility.** `prefers-reduced-motion` disables all animations. Semantic HTML. Skip link. Sufficient contrast on dark background.
- **File size:** Under 30KB.

---

## Voice Notes (ben-tone)

The page content is written in Ben Akiva's personal voice:
- Hebrew is colloquial/spoken: "נראלי", "בסה"כ", "נו כזה", "יאללה"
- English loanwords natural: "דיפולט", "רגקס", "header", "LLM"
- Parenthetical asides: "(אגב, אם יש לך גם ייצוא וואטסאפ...)"
- No "בכבוד רב", no "בברכה", no em-dashes
- Warmth always present, even in technical content
- Self-aware humor: Mailo has personality, admits when it might be wrong
- Exclamation marks for warmth, not aggression
- The user's questions are casual, natural, like someone texting a friend

---

## Quality Checklist

Before delivering, verify:
- [ ] Loads correctly as local HTML file
- [ ] RTL layout works throughout
- [ ] All animations trigger on scroll
- [ ] Typing indicators appear before first 3-4 Mailo answers
- [ ] Sieve funnel animates with stagger
- [ ] Pipeline lights up sequentially (right-to-left in RTL)
- [ ] Progress bar tracks scroll position
- [ ] Mobile responsive (test at 375px)
- [ ] `prefers-reduced-motion` disables animations
- [ ] No horizontal scroll at any viewport
- [ ] Text readable (contrast on dark background)
- [ ] Under 30KB total
- [ ] Commands and file paths display LTR within RTL context
- [ ] Timestamps progress 1:47 to 2:23
- [ ] Step numbers (1-8) are visually clear within the flow
- [ ] The page reads as a practical guide, NOT a feature list
