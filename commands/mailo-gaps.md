---
description: Enter gap-filling mode — answer Mailo's open questions one by one
allowed-tools: Read, Write, Edit
---

Enter Mailo's gap-filling mode. Load all open gaps from state/gaps.json.

1. Show a summary: total open gaps by priority (critical, high, medium, low)
2. Sort by priority (critical first), then by connections_it_would_unlock count
3. Present ONE question at a time with full context:
   - What source raised the question
   - What era and people it relates to
   - What the answer would unlock
4. After the user answers:
   - Update the gap entry (status = "answered", record the answer)
   - Update relevant state files (people.json, timeline.json, etc.)
   - Check if the answer resolves any OTHER open gaps
   - Present the next question
5. If the user says "enough", "stop", or "skip" — respect it

Never present more than one question at a time. Always explain WHY you're asking.

Read the gap skill for full instructions: @${CLAUDE_PLUGIN_ROOT}/skills/gap/SKILL.md
