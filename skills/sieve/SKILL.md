---
name: sieve
description: >
  This skill should be used when processing an email archive (MBOX file) through Mailo's
  5-pass extraction pipeline. Trigger when the user says "run sieve", "process my emails",
  "classify my emails", "continue sieve", or when the ingest skill detects an MBOX file.
  Runs 5 sequential passes: automated emails, spam/marketing, newsletters, transactional,
  and LLM content classification, followed by statistical era detection.
version: 0.1.0
---

# Sieve — 5-Pass Email Extraction Pipeline

## Description

The core email processing engine. Takes a parsed email archive (MBOX) and runs 5 sequential extraction passes, each targeting a specific content type. Every email is classified, every signal is extracted, every layer of intelligence is captured before anything is marked for removal. This is the pipeline that turns a raw inbox into structured biographical data.

**Core principle: Statistics find the signal. LLMs tell the story.** Passes 1-4 use pure code (regex, pattern matching, domain lists) at zero AI cost. Pass 5 uses a cheap LLM only for the remaining human communications. Pass 5b uses statistical change-point detection for era labeling.

## Trigger Conditions

- Activated by the **Ingest** skill when an MBOX file is detected
- User says "run sieve", "process my emails", "classify my emails"
- User says "continue sieve" (to resume an interrupted run)

## Prerequisites

- An MBOX file in `inbox/`
- Python available for email parsing and database operations
- For Pass 5: API access to a cheap LLM (Claude Haiku or GPT-4o-mini)

## Step-by-Step Instructions

### Phase A: Parse MBOX into SQLite

Before any passes run:
1. Parse the MBOX file using Python's `mailbox` library
2. Create `mailo.db` (SQLite) with an `emails` table:
   - `email_id` (TEXT, primary key — Message-ID header or generated hash)
   - `from_name`, `from_email`, `to_addresses`, `cc_addresses`
   - `date` (ISO 8601), `subject`, `body_text`, `body_html`
   - `headers_raw` (JSON blob of all headers)
   - `content_type` (NULL — filled by passes)
   - `pass_number` (NULL — which pass classified it)
   - `action` (NULL — "keep", "delete", or "review")
   - `importance` (NULL — 1-10 scale)
   - `era` (NULL — filled by Pass 5b)
   - `signal_extracted` (BOOLEAN, default false)
   - `classification_reasoning` (TEXT)
3. Report: "Parsed [N] emails from MBOX. Date range: [earliest] to [latest]. Ready to sieve."

### Phase B: The 5-Pass Sieve

#### Pass 1 — Automated / Machine Emails ($0, pure regex)

**Detection rules** (no AI needed):
- `from_email` contains: `noreply`, `no-reply`, `mailer-daemon`, `notifications`, `alert`, `digest`
- `from_email` domain is known automated sender: `github.com`, `vercel.com`, `accounts.google.com`, `facebookmail.com`, `linkedin.com`, `notifications@`, `support@`, `info@`, `team@`
- Body contains "unsubscribe" link (case-insensitive)
- Body contains "This is an automated message" or "Do not reply to this email"
- Body matches known service notification patterns (password reset, verification code, shipping notification)

**Signal extraction** (write to `signal-log.json`):
- Service registration dates ("Welcome to [service]" → registration timestamp)
- Account creation dates
- Platform usage periods (first and last email from a service = usage window)
- Subscription starts/ends
- Shipping notifications → purchase dates and addresses
- Password resets → active account indicators

**Database update**: `content_type = "automated"`, `action = "delete"`, `pass_number = 1`, `signal_extracted = true`

**Expected volume**: ~30-40% of all emails

#### Pass 2 — Spam / Marketing ($0, regex + domain lists)

**Detection rules**:
- Known marketing platforms: `mailchimp.com`, `sendinblue.com`, `constantcontact.com`, `hubspot.com`, `mailgun.com`, `sendgrid.net`, `amazonses.com`
- Subject patterns (case-insensitive): `% off`, `sale`, `limited time`, `exclusive offer`, `free trial`, `act now`
- Bulk political emails: campaign senders, party mailing lists
- Repeated identical senders with promotional content (same sender, 10+ emails, no replies from user)
- Coupon/deal aggregators

**Signal extraction**:
- Political interests/periods (which mailing lists, when subscribed/unsubscribed)
- Brand affinities (which stores, services)
- Entertainment preferences (concert tickets, streaming services)
- Purchase patterns (retail marketing = shopping habits)

**Database update**: `content_type = "marketing"`, `action = "delete"`, `pass_number = 2`

**Expected volume**: ~10-15%

#### Pass 3 — Newsletters / Mailing Lists ($0, pattern matching)

**Detection rules**:
- `List-Unsubscribe` header present
- Sender appears 5+ times with similar subject patterns
- Known newsletter/mailing list senders (alumni associations, organizational updates, community bulletins)
- Consistent periodic sending pattern (weekly, monthly)
- One-to-many sending pattern (same email to many recipients)

**Signal extraction**:
- Community memberships and their active periods
- Organizational affiliations over time
- Engagement periods (when did the user join/leave this community?)
- Topics of interest by period

**Database update**: `content_type = "newsletter"`, `action = "delete"`, `pass_number = 3`

**Exception**: Flag newsletters with high emotional significance (e.g., post-Oct 7 alumni updates). These get `action = "review"` instead of `"delete"`.

**Expected volume**: ~5-10%

#### Pass 4 — Transactional / Receipts / Logistics ($0, pattern matching)

**Detection rules**:
- From known transactional domains: `booking.com`, `airbnb.com`, `hotels.com`, `elal.co.il`, `wizzair.com`, `paypal.com`, `paybox.me`, `buyme.co.il`, `amazon.com`, `ebay.com`
- Subject contains (Hebrew or English): `confirmation`, `receipt`, `booking`, `order`, `invoice`, `reservation`, `אישור`, `הזמנה`, `קבלה`, `הזמנתך`
- Flight confirmations, hotel bookings, purchase receipts
- Campus/office print notifications
- Medical appointments, insurance correspondence
- Bank/financial notifications
- Delivery tracking

**Signal extraction** (this layer is RICH — the skeleton of the life timeline):
- Complete travel timeline (flights, hotels, destinations with dates)
- Purchase history (what, when, how much)
- Addresses (home, work, visited locations)
- Phone numbers
- Medical providers and appointment dates
- Financial services used
- Educational institutions and enrollment periods

**Database update**: `content_type = "transactional"`, `action = "delete"`, `pass_number = 4`

**Exception**: Some transactional emails have sentimental value (e.g., planning a trip for a parent). Flag these for review.

**Expected volume**: ~10-15%

#### Pass 5 — Content Classification (~$0.50, cheap LLM via batch API)

**Target**: Remaining ~30-40% of emails (the real human communications)

**Method**: Send each remaining email through a cheap LLM (Claude Haiku or GPT-4o-mini) with this prompt:

```
You are classifying an email from a personal email archive. The archive owner is an Israeli man, born 1991, based in Tel Aviv. Emails span 2007-2026 and include Hebrew and English.

Classify this email into ONE category:
- professional_logistics: work scheduling, task coordination, file sharing for a job
- organizational: group planning for programs, camps, events, delegations
- personal: emotional content, friendship, family, personal matters, deep correspondence
- self_archive: user emailing themselves files, notes, or documents for safekeeping
- creative_work: designs, presentations, articles, creative content the user made
- study_materials: homework, lecture notes, academic exchanges, exam prep
- file_share: just a file being sent with minimal/no meaningful text

Also assess:
- importance (1-10): How significant is this email for a biographical life record?
- emotional_weight (true/false): Does this email carry emotional significance?
- gem_candidate (true/false): Does this contain a joke, beautiful quote, surprising fact, or meaningful fragment worth preserving?

Return JSON only:
{
  "category": "...",
  "confidence": 0.0-1.0,
  "one_line_summary": "...",
  "importance": 1-10,
  "emotional_weight": true/false,
  "gem_candidate": true/false,
  "gem_fragment": "..." // only if gem_candidate is true — the actual fragment, in original language
}

EMAIL:
From: {from}
To: {to}
Date: {date}
Subject: {subject}
Body: {body_first_1000_chars}
```

**Database update**: Set `content_type`, `importance`, `pass_number = 5`.
- If `importance >= 7` OR `emotional_weight = true` → `action = "keep"`
- If `importance <= 3` AND category is `file_share` or `study_materials` → `action = "delete"`
- Everything else → `action = "review"`

**Gem extraction**: If `gem_candidate = true`, write to `state/gems.json`:
```json
{
  "id": "gem-NNN",
  "fragment": "the actual text, in original language",
  "source_email_id": "email-xxx",
  "type": "quote|joke|fact|moment",
  "date": "2014-06-15",
  "people_involved": ["person-001"],
  "context": "One-line description of where this came from"
}
```

#### Pass 5b — Era Labeling ($0, code only)

**Method**: After all sieve passes complete:
1. Aggregate metadata into monthly time series: email volume, unique contacts, domain diversity, self-sent ratio
2. Run PELT change-point detection (Python `ruptures` library) to find 5-15 natural breakpoints
3. Label each detected segment as a provisional era
4. Cross-reference breakpoints with signal-log events (job changes, travel, service registrations) to name eras
5. Update `era` column for every email
6. Write detected eras to `state/timeline.json`

**Fallback**: If `ruptures` is not available, fall back to simple year-based segmentation.

### Phase C: Post-Sieve Report

After all passes complete, report to user:
```
🔬 Sieve Complete
──────────────────
Total emails processed: [N]
Date range: [earliest] — [latest]

Pass breakdown:
  Pass 1 (Automated):     [N] emails ([X]%)
  Pass 2 (Marketing):     [N] emails ([X]%)
  Pass 3 (Newsletters):   [N] emails ([X]%)
  Pass 4 (Transactional): [N] emails ([X]%)
  Pass 5 (Classified):    [N] emails ([X]%)

Actions:
  Keep:   [N] emails
  Delete: [N] emails
  Review: [N] emails

Eras detected: [N]
Signals extracted: [N]
Gems found: [N]
New people discovered: [N]
Gap questions generated: [N]

Ready for review.
```

## State Files

| File | Access |
|------|--------|
| `state/meta.json` | READ + WRITE (update processing_log, stats) |
| `state/signal-log.json` | WRITE (all extracted signals from passes 1-4) |
| `state/gems.json` | WRITE (gem fragments from pass 5) |
| `state/people.json` | WRITE (new people discovered in emails) |
| `state/timeline.json` | WRITE (eras from pass 5b, events from signals) |
| `state/gaps.json` | WRITE (questions raised during processing) |
| `mailo.db` | CREATE + READ + WRITE (the email database) |

## Rules and Constraints

- **NEVER delete emails from Gmail.** Only classify and recommend actions.
- **Preserve ALL signals before marking anything for deletion.** Extract first, classify second.
- **If a pass is uncertain about an email, leave it for the next pass.** Better to under-classify than misclassify.
- **Log every classification decision with reasoning** in `classification_reasoning` column.
- **Hebrew content**: Classify based on meaning, preserve original text in all fields.
- **Idempotent**: If sieve is interrupted and resumed, skip already-classified emails (check `pass_number IS NOT NULL`).
- **Batch API for Pass 5**: Use batch/async API to minimize cost. Target: $0.50 or less for 3,686 emails' remaining ~30%.
- **Never process more than one MBOX at a time** without user confirmation.

## Output Format

The primary output is the populated `mailo.db` SQLite database and updated state files. The user-facing output is the post-sieve report shown above.
