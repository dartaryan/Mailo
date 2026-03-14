---
name: gap
description: >
  This skill should be used when working with Mailo's open questions system. Trigger
  when the user says "what questions do you have?", "what don't you know?",
  "gap filling mode", "ask me questions", "show me open gaps", "what's unresolved?",
  or when any other skill encounters missing or ambiguous information. Also trigger
  when new data arrives that might auto-resolve existing gaps.
version: 0.1.0
---

# Gap — Open Questions Tracker

## Description

Manages open questions — things Mailo doesn't know but wants to. When any skill encounters missing or ambiguous information, it writes a gap question here. The gap system is what makes Mailo a living, evolving intelligence engine rather than a static archive. Gaps drive the conversation forward: every answer unlocks new connections, resolves ambiguities, and deepens the life record.

## Trigger Conditions

- Any skill encounters missing or ambiguous information
- User says "what questions do you have?" or "what don't you know?"
- User says "gap filling mode" or "ask me questions"
- New data arrives → check against existing gaps for auto-resolution
- User says "show me open gaps" or "what's unresolved?"

## Step-by-Step Instructions

### 1. Gap Schema

```json
{
  "id": "gap-001",
  "question": "Who is Ahud whose mother died in June 2012? Maya said 'she was my teacher.'",
  "generated_from": "email-xxx",
  "generated_by": "sieve",
  "date_generated": "2026-03-15",
  "priority": "medium",
  "connections_it_would_unlock": ["person-001", "era-06"],
  "tags": ["person_identification", "szarvas_2012"],
  "status": "open",
  "answer": null,
  "answered_date": null,
  "source_of_answer": null
}
```

### 2. Priority Levels

- **critical**: Blocks understanding of a major era or relationship (e.g., "Who is this person in 50+ emails?")
- **high**: Would unlock significant connections (e.g., "Why did this relationship end?")
- **medium**: Would enrich an existing entry (e.g., "Is this person the same as that person?")
- **low**: Nice to know but not blocking anything (e.g., "What was this attachment about?")

### 3. Generating Gaps

Any skill can generate a gap by writing to `state/gaps.json`. When creating a gap:
1. Write a clear, specific question (not vague)
2. Include context: what source raised the question, what era/people it relates to
3. Assign a priority based on what the answer would unlock
4. Tag with relevant categories for grouping
5. List the specific `connections_it_would_unlock` (person_ids, era_ids, etc.)

Good gap questions:
- "Who is [specific name]? They appear in [N] emails from [specific period] in the context of [specific activity]."
- "Maya mentions 'Ahud's mother died' in June 2012. Who is Ahud? This would help understand Maya's emotional state in Era 6."

Bad gap questions:
- "Tell me more about this person." (too vague)
- "What happened?" (no context)

### 4. Auto-Resolution

When new data arrives (new emails processed, user provides info, new source ingested):
1. Load all `status: "open"` gaps
2. Check if the new data contains information that answers any gap
3. If a likely answer is found:
   - Update the gap: `status = "auto_resolved"`, `answer = [found info]`, `source_of_answer = [source]`
   - Present to user for confirmation: "I think I found the answer to: [question]. Based on [source], it looks like [answer]. Is that right?"
   - If confirmed → update relevant state files
   - If rejected → revert to `status: "open"`, note the failed auto-resolution

### 5. Gap-Filling Mode

When the user activates this mode:
1. Load all open gaps from `state/gaps.json`
2. Sort by priority (`critical` → `high` → `medium` → `low`), then by `connections_it_would_unlock` count (descending)
3. Present ONE question at a time with full context:
   - "While processing emails from [era], I found [N] emails mentioning [person/event]."
   - "[The specific question]"
   - "Knowing the answer would help me [what it unlocks]."
4. After user answers:
   - Update the gap entry: `status = "answered"`, `answer = [user's response]`, `answered_date = [now]`, `source_of_answer = "user"`
   - Update relevant state files with the new information (people.json, timeline.json, etc.)
   - Check if the answer resolves any OTHER open gaps
   - Present next question
5. User can say "enough", "stop", or "skip" to exit or skip

### 6. Gap Statistics

Track and report:
- Total open gaps by priority
- Gaps resolved this session
- Gaps that new data could answer
- Oldest unresolved gap

## State Files

| File | Access |
|------|--------|
| `state/gaps.json` | READ + WRITE (primary data store) |
| `state/people.json` | READ + WRITE (update when gap answers reveal people info) |
| `state/timeline.json` | READ + WRITE (update when gap answers reveal events/eras) |
| `state/gems.json` | READ + WRITE (sometimes a gap answer reveals a gem) |
| `state/meta.json` | WRITE (update total_open_gaps stat) |

## Rules and Constraints

- **Never present more than one question at a time** — let the user focus
- **Always explain WHY you're asking** — what the answer would unlock
- **If the user says "I don't know" or "skip"**: Mark as `status: "skipped"` and move on. Don't push.
- **Don't ask the same question twice** — check `answered` and `skipped` status before presenting
- **Related gaps should be grouped**: If gaps are about the same person or era, present them in sequence
- **Respond in English** — gap questions are phrased in English. Hebrew names, quotes, and subjects are preserved within the question.
- **Gaps are never deleted** — even answered gaps remain in the log for provenance tracking

## Output Format

When presenting gaps to the user:
```
❓ Open Questions: [N] total
   Critical: [N] | High: [N] | Medium: [N] | Low: [N]

Next question (priority: [level]):
  [The question with full context]
  This would help: [what it unlocks]
```

When presenting gap-filling summary:
```
📋 Gap-Filling Session Complete
────────────────────────────────
Questions asked: [N]
Answered: [N]
Skipped: [N]
Auto-resolved by answers: [N]
Remaining open: [N]
```
