---
description: View, query, or update the life timeline (eras and events)
allowed-tools: Read, Write, Edit, Grep
argument-hint: [year/period or "all"]
---

Work with Mailo's life timeline.

If $ARGUMENTS contains a year or date range: filter eras and events to that period. Show matching eras with their people, events, and themes.

If $ARGUMENTS is "all" or empty: show the full timeline — all eras in chronological order with key people, events, themes, and summaries.

If the user wants to correct era boundaries, names, or split/merge eras, guide them through the change and update state/timeline.json.

Present in the timeline output format defined in the skill.

Read the timeline skill for full instructions: @${CLAUDE_PLUGIN_ROOT}/skills/timeline/SKILL.md
