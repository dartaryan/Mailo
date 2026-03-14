---
name: timeline
description: >
  This skill should be used when working with Mailo's life timeline — eras and events.
  Trigger when the user says "show me my timeline", "what happened in [year/period]?",
  "what eras do you have?", or when era detection completes after sieve. Also trigger
  when the user corrects era boundaries or names, or asks about events in a specific
  time period.
version: 0.1.0
---

# Timeline — Era Detection and Event Placement

## Description

Manages the life timeline — the chronological structure of eras (life periods) and events. Eras are detected statistically from email metadata patterns and refined through user confirmation. Events are placed on the timeline as they are discovered during analysis. Together, eras and events form the biographical scaffolding of the life record.

## Trigger Conditions

- Sieve completes (Pass 5b era detection)
- New events discovered during analysis
- User asks "what happened in [year/period]?"
- User asks "show me my timeline"
- User corrects era boundaries or names
- User says "what eras do you have?"

## Step-by-Step Instructions

### 1. Era Detection (Automated via Pass 5b)

After sieve completes:
1. Aggregate email metadata into monthly time series: volume, unique contacts, domain diversity, self-sent ratio
2. Run PELT change-point detection (`ruptures` library) to find 5-15 natural breakpoints
3. Label each segment as a provisional era
4. Cross-reference breakpoints with signal-log events (job changes, travel, service registrations) to propose era names
5. Write to `state/timeline.json` with `status: "inferred"`

### 2. Era Schema

```json
{
  "id": "era-01",
  "name": "Havayah 2008 + Genesis Preparation",
  "date_from": "2007-10",
  "date_to": "2008-06",
  "auto_detected": true,
  "confirmed": false,
  "people_ids": ["person-001", "person-015"],
  "key_events": ["event-001", "event-002"],
  "themes": ["Jewish youth programming", "creative recognition", "diaspora connections"],
  "summary": "Entry into Jewish youth programming. First overnight design work. Genesis application.",
  "email_count": 45,
  "status": "inferred"
}
```

### 3. Event Schema

```json
{
  "id": "event-001",
  "type": "milestone",
  "name": "First published article (JVibe)",
  "date": "2009-08",
  "era_id": "era-04",
  "people_ids": ["person-055"],
  "description": "Ben's first published article in a Jewish teen magazine.",
  "significance": 5,
  "source_email_ids": ["email-xxx"],
  "status": "confirmed"
}
```

### 4. Event Types

- `milestone` — Achievement or significant first
- `travel` — Trip, relocation, or extended stay
- `job_change` — New job, role change, or end of employment
- `relationship_change` — New relationship, breakup, reconciliation
- `creative_work` — Published work, design project, performance
- `education` — Enrollment, graduation, program completion
- `loss` — Death, separation, ending
- `celebration` — Birthday, wedding, achievement celebration
- `conflict` — Disagreement, crisis, challenge
- `identity_moment` — Coming out, major decision, self-discovery

### 5. Operations

- **Auto-detect eras**: Run PELT on email metadata → propose era boundaries
- **Name eras**: Cross-reference detected boundaries with signal-log (travel dates, job changes, significant contacts appearing/disappearing)
- **Place events**: Every significant find from sieve or deep analysis → create event entry
- **Connect events to people**: Link event → people involved
- **Timeline queries**: "What happened between 2014 and 2016?" → filter eras + events by date range
- **Validate with user**: Present detected eras and ask: "Does this match your memory? Where should I adjust boundaries?"
- **Merge eras**: If user says two detected segments are really one era
- **Split eras**: If user says a detected segment should be two eras

## State Files

| File | Access |
|------|--------|
| `state/timeline.json` | READ + WRITE (primary data store) |
| `state/people.json` | READ (to link people to eras) + WRITE (update era_appearances) |
| `state/signal-log.json` | READ (to name and validate eras) |
| `state/gaps.json` | WRITE (questions about unclear periods) |
| `state/meta.json` | WRITE (update total_eras and total_events stats) |

## Rules and Constraints

- **Auto-detected eras are `status: "inferred"`** until the user confirms them
- **Era boundaries can overlap** — transition periods are natural (e.g., ending one job while starting another)
- **Some themes span multiple eras** — recurring activities like "Szarvas camp" appear in 8+ eras. These are recurring themes, NOT standalone eras (unless the activity defines the entire period)
- **Travel events go in the era they occurred in**, not as standalone eras — unless the trip was long enough to define a period (e.g., a 4-month backpacking trip)
- **IDs are permanent** — never reuse an era_id or event_id
- **User corrections override auto-detection**: If the user says "that's not right, it was actually...", update and set `status: "corrected"`
- **Respond in English** — era names and summaries are in English. Hebrew names for programs, places, and people are preserved as-is.

## Output Format

When presenting the timeline to the user:
```
📅 Life Timeline
═══════════════════

ERA 1: [name] ([date_from] — [date_to]) [status]
  Key people: [names]
  Key events: [event names]
  Themes: [theme list]
  Summary: [one paragraph]

ERA 2: [name] ([date_from] — [date_to]) [status]
  ...

──────────────────
Total: [N] eras, [N] events
Date range: [earliest] — [latest]
```
