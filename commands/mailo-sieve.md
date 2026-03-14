---
description: Run the 5-pass email extraction pipeline on an MBOX archive
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

Run Mailo's 5-pass sieve on the MBOX file in `inbox/`.

Phase A: Parse the MBOX into SQLite (mailo.db) if not already done.

Phase B: Run passes sequentially:
- Pass 1: Automated/machine emails (regex, $0)
- Pass 2: Spam/marketing (regex + domain lists, $0)
- Pass 3: Newsletters/mailing lists (pattern matching, $0)
- Pass 4: Transactional/receipts (pattern matching, $0)
- Pass 5: Content classification (cheap LLM, ~$0.50)
- Pass 5b: Era labeling (PELT change-point detection, $0)

Extract signals from every pass before classifying. Update state files (signal-log.json, gems.json, people.json, timeline.json, gaps.json).

Phase C: Report results — total processed, pass breakdown, action counts, eras detected, signals extracted.

Read the sieve skill for full instructions: @${CLAUDE_PLUGIN_ROOT}/skills/sieve/SKILL.md
