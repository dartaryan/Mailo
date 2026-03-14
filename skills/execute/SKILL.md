---
name: execute
description: >
  This skill should be used when applying Mailo's classifications back to Gmail.
  Trigger when the user says "apply labels", "clean up Gmail", "execute",
  "run the labels", "delete the emails marked for deletion", or "label my emails".
  This is the ONLY skill that touches Gmail. Never triggered automatically —
  always requires explicit user request.
version: 0.1.0
---

# Execute — Gmail Label/Delete Operations

## Description

The ONLY skill that touches Gmail. Applies classifications back to the real mailbox — labels for emails to keep, a `_TO_DELETE` label for emails to discard. This skill is the bridge between Mailo's intelligence and the user's actual inbox cleanup. It is NEVER triggered automatically — the user must explicitly request execution.

**Core safety principle**: Mailo never permanently deletes anything. It labels emails for deletion; the user performs the actual delete from Gmail.

## Trigger Conditions

- User says "apply labels", "clean up Gmail", "execute", "run the labels"
- User says "delete the emails marked for deletion"
- User says "label my emails"
- **NEVER triggered automatically** — always requires explicit user request

## Prerequisites

Before executing, verify ALL of the following:
1. Gmail API access is available (via MCP connector or OAuth)
2. All target emails have `action = "keep"` or `action = "delete"` confirmed by user
3. A label taxonomy has been established
4. The user has reviewed and approved the classification summary

## Step-by-Step Instructions

### 1. Label Convention

Follow this hierarchical pattern: `Category/SubCategory/Year/Type`

Types:
- `Memories` — sentimental, personal, emotionally significant
- `Logistics` — practical, date-establishing, transactional

Examples:
- `Life/Szarvas/2014/Memories`
- `Life/Diller/2009/Logistics`
- `People/Gal/Love`
- `Life/Travel/Euro-Trip-2014`
- `Self/Notes`
- `Work/[Company]/[Year]`

### 2. Apply Labels Operation

1. Read from state (mailo.db or equivalent) all emails where `action = "keep"` and a label is assigned
2. Group by label path
3. Present to user:
   ```
   Ready to apply labels:
   ─────────────────────
   Life/Szarvas/2014/Memories    → 23 emails
   Life/Diller/2009/Logistics    → 15 emails
   People/Gal/Love               → 8 emails
   ...
   Total: [N] labels across [M] emails

   Proceed? (yes/no)
   ```
4. On confirmation:
   - Create labels in Gmail if they don't exist
   - Apply labels to each email using Gmail API
   - Process in batches of 100 max per API call
5. Report:
   ```
   ✅ Labels Applied
   ─────────────────
   Labels created: [N] new
   Emails labeled: [M]
   Errors: [N] (if any — list which emails failed)
   ```

### 3. Delete (Label for Deletion) Operation

1. Read all emails where `action = "delete"`
2. Present to user:
   ```
   📛 [N] emails marked for deletion.

   This will apply the '_TO_DELETE' label (NOT permanent deletion).
   You can review in Gmail: search 'label:_TO_DELETE'
   Then select all and delete manually.

   Breakdown:
     Automated/machine: [N]
     Marketing/spam: [N]
     Newsletters: [N]
     Transactional: [N]
     Low-importance: [N]

   Proceed? (yes/no)
   ```
3. On confirmation:
   - Create `_TO_DELETE` label in Gmail if it doesn't exist
   - Apply `_TO_DELETE` label to all target emails
   - Process in batches of 100
4. Report:
   ```
   🏷️ Deletion Labels Applied
   ───────────────────────────
   Emails labeled '_TO_DELETE': [N]

   Next step: Go to Gmail, search 'label:_TO_DELETE',
   select all, and delete. This is YOUR action — Mailo
   does not permanently delete.
   ```

### 4. Operation Log

Every execution creates a log entry in `state/meta.json` → `processing_log`:
```json
{
  "operation": "apply_labels",
  "timestamp": "2026-03-15T14:30:00Z",
  "emails_affected": 150,
  "labels_created": 12,
  "errors": 0,
  "details": "Applied labels to 150 emails across 12 label paths"
}
```

## State Files

| File | Access |
|------|--------|
| `state/meta.json` | READ + WRITE (processing_log, stats) |
| `mailo.db` | READ (email classifications, actions, labels) |

## Rules and Constraints

- **NEVER permanently delete emails.** Only apply `_TO_DELETE` label. The user deletes manually from Gmail.
- **NEVER execute without explicit user confirmation** in the chat. Show exactly what will happen and wait for "yes".
- **NEVER modify an email that doesn't have a confirmed action.** Emails with `action = "review"` must be resolved first.
- **Process in batches of 100 max** per Gmail API call to respect rate limits
- **If any operation fails**, report the failure, log it, and continue with remaining emails. Never silently skip failures.
- **Log every operation**: what was done, when, how many emails, any errors
- **Idempotent**: If the same label operation is run twice, it should not create duplicate labels or errors
- **Respond in English** — all reports and confirmations are in English

## Output Format

Pre-execution summary → User confirmation → Execution report → Next steps recommendation
