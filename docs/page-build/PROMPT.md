# Claude Code Prompt — Build the Mailo Landing Page

Copy and paste everything below this line into Claude Code:

---

## Task

Build a single-file HTML landing page for Mailo (a biographical email intelligence agent). The page will be deployed on GitHub Pages as `index.html`.

## Before you write any code

Read these two files carefully — they contain everything you need:

1. `docs/page-build/design-brief.md` — full design specifications: color palette, typography, layout, animations, technical constraints, and quality checklist
2. `docs/page-build/content-hebrew.md` — the exact Hebrew content for every element on the page, structured as a chat conversation with markup annotations

Also glance at `docs/how-to-work-with-mailo.html` — this is the OLD version of the page that we're replacing. The new design is completely different.

## Key concept

The page is a **practical step-by-step guide** that walks someone through the 8 stages of working with Mailo. It's formatted as a late-night chat conversation — a user asking "how does this work?" and Mailo explaining each step in order. Dark background, chat bubbles sliding in on scroll, typing indicators, a funnel animation for the sieve, and a pipeline animation showing the full journey upfront.

**IMPORTANT: This is NOT a feature list or product page. It's a walkthrough of the actual working process.** Each step explains what happens, who does what (Mailo vs. the user), and what the output is.

## Critical requirements

- Single HTML file, no build step, no frameworks, no external JS libraries
- Hebrew RTL throughout (`dir="rtl"` on html)
- Command names like `/mailo-sieve` and paths like `inbox/` display LTR within RTL context
- All animations are CSS-only, triggered by IntersectionObserver in vanilla JS
- `prefers-reduced-motion` disables all animations
- Mobile-first responsive design (this is a chat page — phones are its natural home)
- Google Fonts loaded externally (Heebo + a monospace font like JetBrains Mono or Fira Code)
- Under 30KB total file size
- The typing indicator (bouncing dots) appears before the first 3-4 Mailo answers only
- The pipeline animation has 8 nodes (not 7) — see content file for labels
- The pipeline animates right-to-left (RTL direction)

## Output

Create the file at the project root as `index.html`. After building, open it in the browser to verify it works.
