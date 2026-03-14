---
description: Apply Gmail labels and mark emails for deletion (requires confirmation)
allowed-tools: Read, Write, Edit, Bash
---

Apply Mailo's email classifications to Gmail. This is a DESTRUCTIVE operation that requires explicit user confirmation at every step.

SAFETY RULES:
- NEVER permanently delete emails — only apply the _TO_DELETE label
- NEVER execute without showing exactly what will happen and getting "yes"
- NEVER modify emails that don't have a confirmed action

Steps:
1. Check prerequisites: Gmail API access, confirmed classifications
2. Show the user exactly what will happen (label counts, deletion counts)
3. Wait for explicit confirmation
4. Execute in batches of 100
5. Report results including any errors

Read the execute skill for full instructions: @${CLAUDE_PLUGIN_ROOT}/skills/execute/SKILL.md
