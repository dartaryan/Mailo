---
description: Run deep biographical analysis — patterns, connections, insights
allowed-tools: Read, Write, Edit, Grep
argument-hint: [topic or "all"]
---

Run Mailo's deep analysis engine at Depth Levels 5-9.

Prerequisites: verify at least 3 eras in state/timeline.json and at least 10 people in state/people.json. If prerequisites aren't met, explain what's needed.

If $ARGUMENTS contains a topic (e.g., "relationships", "mentors", "career"): focus analysis on that domain.

If $ARGUMENTS is "all" or empty: run a broad analysis across all available data.

Analysis levels:
- Level 5 (Deep): Behavioral coding over time
- Level 6 (Indirect): What actions imply beyond surface meaning
- Level 7 (Very Deep): Archetypal role identification
- Level 8 (Surprising): Hidden network connections
- Level 9 (Recurring): Cross-domain pattern synthesis

Every insight must trace back to specific evidence. Present insights gently — these are about a real person's life. Mark all observations as [Analyst Note].

NEVER present Level 7-9 insights without Level 1-6 facts confirmed first.

Read the insight skill for full instructions: @${CLAUDE_PLUGIN_ROOT}/skills/insight/SKILL.md
