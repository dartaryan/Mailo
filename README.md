# Mailo — Biographical Intelligence Engine

Mailo processes personal email archives (and other life data) into structured biographical intelligence. It extracts people, eras, events, behavioral patterns, and deep psychological insights from communication data spanning years or decades.

**Core insight**: Statistics find the signal. LLMs tell the story. 70-80% of insights come from metadata alone. The LLM handles the remaining 20-30% that requires understanding language, tone, and meaning.

## What Mailo Does

1. **Ingests** data sources (MBOX archives, WhatsApp exports, screenshots, PDFs, etc.)
2. **Sieves** email archives through a 5-pass extraction pipeline — from zero-cost regex passes to targeted LLM classification
3. **Builds** a people directory with entity resolution and relationship tracking
4. **Detects** life eras using statistical change-point analysis on email metadata
5. **Tracks** open questions and fills gaps through interactive Q&A
6. **Generates** deep biographical insights at 9 depth levels
7. **Executes** Gmail cleanup — labels for keepers, deletion labels for the rest

## Commands

| Command | Description |
|---------|-------------|
| `/mailo-status` | Dashboard showing current processing state |
| `/mailo-ingest` | Process new data from the inbox/ folder |
| `/mailo-sieve` | Run the 5-pass email extraction pipeline |
| `/mailo-people` | Query or update the people directory |
| `/mailo-timeline` | View or update the life timeline |
| `/mailo-gaps` | Enter gap-filling mode (interactive Q&A) |
| `/mailo-insight` | Run deep biographical analysis |
| `/mailo-execute` | Apply labels to Gmail (requires confirmation) |

## Skills

| Skill | Purpose |
|-------|---------|
| `ingest` | Data router — identifies input type, activates processing chain |
| `sieve` | 5-pass email extraction pipeline |
| `people` | People directory with entity resolution |
| `timeline` | Era detection and event placement |
| `gap` | Open questions tracker and gap-filling mode |
| `insight` | Deep analysis engine (Depth Levels 5-9) |
| `execute` | Gmail label/delete operations |

## Data Storage

State files in `state/` persist between sessions:

- `meta.json` — project metadata, owner profile, settings, stats
- `people.json` — people directory
- `timeline.json` — eras and events
- `gems.json` — preserved meaningful fragments (quotes, jokes, moments)
- `gaps.json` — open questions
- `insights.json` — behavioral patterns and deep analysis
- `signal-log.json` — extracted intelligence from sieve passes

The `mailo.db` SQLite database is created when the first MBOX is processed.

## Setup

1. Install the plugin in Cowork
2. Place your MBOX file (from Google Takeout) in the `inbox/` folder
3. Run `/mailo-status` to verify, then `/mailo-ingest` to begin

## The 9 Depth Levels

Mailo analyzes data at increasing depth:

**Surface (1-3)**: What happened? Basic facts, contacts, timelines. Free — pure code.

**Analytical (4-6)**: What does it mean? Behavioral patterns, relationship evolution, implied significance. Pennies per insight.

**Profound (7-9)**: What deep truth does this reveal? Archetypal roles, hidden networks, cross-domain life patterns. Targeted LLM analysis on the most significant relationships only.
