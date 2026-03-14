---
name: insight
description: >
  This skill should be used when performing deep biographical analysis at Mailo's
  Depth Levels 5-9. Trigger when the user says "what patterns do you see?",
  "tell me about my [relationship/behavior/pattern]", "what's the deepest thing
  you've noticed?", "analyze my relationships", "find patterns", or when 3+ eras
  have been fully processed. Produces behavioral patterns, relationship dynamics,
  identity evolution, hidden connections, and cross-domain recurring themes.
version: 0.1.0
---

# Insight — Deep Analysis Engine (Depth Levels 5-9)

## Description

The deep analysis engine — where Mailo becomes biographical intelligence rather than a filing system. This skill operates at Depth Levels 5-9, finding behavioral patterns, relationship dynamics, identity evolution, hidden network connections, and cross-domain recurring themes across the user's entire life record. These are the insights that no email client, CRM, or organizer can produce.

**Key principle**: Every insight MUST trace back to specific evidence. Observations are not facts — they are analyst notes, presented with humility and always subject to user correction.

## Trigger Conditions

- Multiple eras have been processed and enough data exists for cross-era analysis (minimum 3 eras)
- User asks "what patterns do you see?" or "tell me about my [relationship/behavior/pattern]"
- User asks "what's the deepest thing you've noticed?"
- User asks about a specific person's significance or a specific pattern
- Triggered automatically after 3+ eras are fully processed and confirmed

## Step-by-Step Instructions

### 1. Prerequisites Check

Before generating insights, verify:
- At least 3 eras are in `state/timeline.json` with `status: "confirmed"` or `"inferred"`
- At least 10 people are in `state/people.json`
- Signal-log has data from completed sieve passes
- **NEVER present Level 7-9 insights without the user having confirmed the underlying Level 1-6 facts first**

### 2. Insight Types

| Type | Description | Example |
|------|-------------|---------|
| `behavioral_pattern` | Recurring behavior across contexts | "Builder-Documenter: you build AND document simultaneously" |
| `relationship_pattern` | How relationships form, evolve, end | "You seek mentor figures who give autonomy" |
| `identity_evolution` | How self-concept changes over time | "Coming out in Nov 2017 is the structural pivot of the timeline" |
| `creative_growth` | How creative output evolves | "From t-shirt designs to full congress staging to digital games" |
| `coping_mechanism` | How stress/loss/conflict is handled | "You process conflict through writing, then share with trusted person" |
| `recurring_theme` | Themes that reappear across eras | "Separations and loss. Difficulty when eras end." |
| `turning_point` | Moments that changed trajectory | "The IDF letter Maya wrote was institutional advocacy at personal cost" |
| `character_trait` | Stable personality characteristics | "Constant comedian, culture explorer, initiator, summarizer" |
| `cross_era_connection` | Hidden links between separate periods | "Szarvas network connects nearly every era of adult life" |
| `hidden_network` | Surprising connections between people | "Person X from era A is related to Person Y from era B" |

### 3. Insight Schema

```json
{
  "id": "insight-001",
  "type": "behavioral_pattern",
  "title": "Builder-Documenter Personality",
  "description": "You don't just build — you document while building. This is visible from age 17 through present. Most builders hate documentation. Most documenters don't build. You do both simultaneously.",
  "depth_level": 7,
  "first_evidence_era": "era-02",
  "supporting_eras": ["era-04", "era-07", "era-10a"],
  "supporting_people": [],
  "supporting_events": ["work-001", "work-002"],
  "supporting_email_ids": [],
  "confidence": "high",
  "status": "inferred",
  "generated_date": "2026-03-15"
}
```

### 4. Analysis Techniques by Depth Level

**Level 5 — Deep**: Behavioral coding.
- Track specific behaviors over time: how does the user address authority figures? How do they handle conflict? When do they use humor?
- Compare early vs. late patterns in the same behavioral category
- Look for changes that correlate with life events

**Level 6 — Indirect**: What actions IMPLY beyond their surface meaning.
- "Maya wrote an IDF letter for you" isn't just logistics — it's institutional advocacy at personal cost
- "You sent the letter to Gal 6 minutes after writing it to Yael" implies partner-as-emotional-processing-partner
- Read between the lines of timing, word choice, and who was CC'd

**Level 7 — Very Deep**: Archetypal role identification.
- Compare all mentor figures across eras — do they share traits?
- Compare all friendships that ended — is there a pattern?
- What role does the user play in relationships (initiator, documenter, connector)?
- Map the recurring cast of character types in the user's life

**Level 8 — Surprising**: Hidden network connections.
- Graph shortest-path between seemingly unrelated people
- "Person A from camp 2010 is connected to Person B from university 2018 through Person C"
- Identify bridge people who connect separate social circles
- Find unexpected overlaps between life domains

**Level 9 — Recurring**: Cross-domain pattern synthesis.
- "The same relational pattern (seek autonomy, build alone, document everything) appears in camp → army → university → work → entrepreneurship"
- "This isn't a career strategy — it's a personality structure"
- Identify the 3-5 deepest truths about the person's life patterns

### 5. Confidence Levels

- **high**: Pattern appears in 3+ separate eras/contexts with direct evidence
- **medium**: Pattern appears in 2 contexts, or in 3+ contexts with indirect evidence
- **speculative**: Single instance with extrapolation, or pattern inferred from absence of evidence

## State Files

| File | Access |
|------|--------|
| `state/insights.json` | READ + WRITE (primary data store) |
| `state/people.json` | READ (relationship data for pattern analysis) |
| `state/timeline.json` | READ (era data for cross-era analysis) |
| `state/signal-log.json` | READ (behavioral signals) |
| `state/gems.json` | READ (meaningful fragments as evidence) |
| `state/gaps.json` | WRITE (new questions raised by analysis) |
| `state/meta.json` | WRITE (update total_insights stat) |

## Rules and Constraints

- **Every insight MUST trace back to specific evidence** — email_ids, person_ids, era_ids. No unsupported claims.
- **Mark all insights as `[Analyst Note]`** in user-facing output — distinguish observation from fact
- **Present insights gently.** These are about a real person's life. Be thoughtful, not clinical.
- **NEVER present Level 7-9 insights without Level 1-6 facts confirmed first.** Build understanding layer by layer.
- **User can reject any insight.** If they say "that's not right", set `status: "rejected"` and note their correction.
- **Don't over-analyze.** Not every email exchange is significant. Focus on patterns that repeat.
- **Respond in English** — insight descriptions are in English. Hebrew quotes used as evidence are preserved in original.
- **Depth levels are earned, not assumed.** Start with Level 5 analysis. Only go deeper when there's sufficient evidence.

## Output Format

When presenting insights:
```
🔮 Insight: [title]
   Depth Level: [N]/9
   Type: [insight_type]
   Confidence: [high/medium/speculative]

   [Analyst Note]
   [The insight description — written with warmth and care]

   Evidence:
   - [Era]: [specific evidence]
   - [Era]: [specific evidence]
   - [Person]: [specific evidence]

   This connects to: [related insights, if any]
```
