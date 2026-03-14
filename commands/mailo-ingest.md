---
description: Process new data — route files from inbox/ through Mailo's pipeline
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Check the `inbox/` directory for files. If files are present, list them with their types and sizes.

For each file, identify its type:
- `.mbox` → email archive, route to sieve skill
- `.txt` with WhatsApp format → WhatsApp export
- `.png`, `.jpg`, `.jpeg` → screenshot/image
- `.json` from Google Takeout → location history
- `.pdf` → document
- `.csv`, `.xlsx` → spreadsheet/data
- `.md` from previous Mailo sessions → migration data

If no files are in `inbox/`, ask the user to provide data (upload a file, paste text, or place files in the inbox folder).

Apply multi-angle extraction to every piece of data:
1. Metadata (dates, times, names, addresses)
2. Content surface (one-line summary)
3. Behavioral signals (time of day, language, formality)
4. Emotional reading (tone, sentiment, warmth vs distance)
5. Relational signals (who is mentioned, implied relationships)
6. What's missing (gap questions → write to state/gaps.json)

Report what was found and which processing chain will be activated.

Read the ingest skill for full instructions: @${CLAUDE_PLUGIN_ROOT}/skills/ingest/SKILL.md
