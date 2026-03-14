---
description: Query, browse, or update the people directory
allowed-tools: Read, Write, Edit, Grep
argument-hint: [person name or "all" or "top"]
---

Work with Mailo's people directory.

If $ARGUMENTS contains a person name: search state/people.json for that person using fuzzy name matching and email matching. Present their full profile including importance, eras, roles, relationships, and connections.

If $ARGUMENTS is "all" or "directory": list all people sorted by importance, showing name, importance score, era count, and email count.

If $ARGUMENTS is "top" or "important": show people with importance >= 4, sorted by era appearances count.

If $ARGUMENTS contains "merge": guide the user through merging two person entries.

If no arguments: show a summary of the people directory — total count, importance distribution, and top 10 by importance.

Read the people skill for full instructions: @${CLAUDE_PLUGIN_ROOT}/skills/people/SKILL.md
