---
description: Show Mailo's current status — what's been processed, what's pending
allowed-tools: Read, Grep
---

Read state/meta.json and all state files to present a comprehensive status report.

Show:
1. Processing status: total emails processed, date range, sources loaded
2. People: total count, importance distribution (how many at each level 1-5)
3. Timeline: total eras (confirmed vs inferred), total events
4. Gaps: open count by priority, recently answered
5. Insights: total generated, by type and depth level
6. Gems: total preserved fragments
7. Pending actions: emails awaiting review, unprocessed files in inbox/

Format as a clean dashboard. If nothing has been processed yet, say so and suggest starting with /mailo-ingest.
