# Mailo Processing Pipeline — Step by Step
> March 13, 2026 | Practical implementation plan

---

## Where Everything Lives

One local folder on your machine:

```
mailo/
├── inbox/                    ← You put MBOX files here
│   ├── dartaryan.mbox
│   └── benakiva1991.mbox
├── db/
│   └── mailo.db              ← SQLite database (single file, grows with each pass)
├── output/
│   ├── master-sheet.xlsx      ← The reviewable spreadsheet (regenerated after each pass)
│   ├── signal-log.json        ← Intelligence extracted from each sieve layer
│   └── gems.json              ← Preserved meaningful fragments
└── scripts/                   ← Processing scripts (Cowork runs these)
```

**The core principle**: One SQLite database is the single source of truth. Every pass reads from it and writes back to it. The spreadsheet is just a human-readable VIEW of the database, regenerated whenever you want to review.

---

## PHASE 1: EXPORT (You do this, ~10 min)

### Step 1.1 — Export dartaryan@gmail.com
- Go to takeout.google.com
- Sign in as dartaryan@gmail.com
- Deselect all → select only "Mail"
- Choose MBOX format
- Export once → .zip → 2GB max
- Download when ready

### Step 1.2 — Export benakiva1991@gmail.com
- Same process, second account

### Step 1.3 — Place files
- Unzip both
- Put the .mbox files in `mailo/inbox/`

**Output**: Two .mbox files sitting in a folder. That's it.

---

## PHASE 2: PARSE (Cowork runs this, ~5 min, $0)

**Goal**: Turn raw MBOX into a database row per email. Pure code, no AI.

### Step 2.1 — Parse every email
For each email in both MBOX files, extract:
- `email_id` (Message-ID header, unique identifier)
- `mailbox` (which account: dartaryan / benakiva1991)
- `date` (parsed to standard format)
- `year` (extracted from date)
- `month` (extracted from date)
- `from_email` (sender address)
- `from_name` (sender display name)
- `to_emails` (recipient addresses, comma-separated)
- `cc_emails` (CC addresses, comma-separated)
- `subject` (original subject line)
- `body_text` (plain text body, stripped of quoted replies)
- `body_length` (character count of body)
- `has_attachment` (true/false)
- `attachment_names` (filenames only, not the actual files)
- `thread_id` (from In-Reply-To / References headers)
- `is_self_sent` (from_email = account owner's email)
- `language` (detected: hebrew / english / mixed — via Unicode range check)
- `domain` (sender's email domain: gmail.com, jdc.org, campus.technion.ac.il, etc.)

### Step 2.2 — Write to database
Create `mailo.db` with table `emails`. One row per email. All fields above as columns. Add these processing columns (empty for now):
- `content_type` (null — will be filled by sieve passes)
- `importance` (null — will be filled later)
- `action` (null — will be "keep" or "delete")
- `label` (null — Gmail label recommendation)
- `people_ids` (null — links to people found later)
- `era` (null — life period, detected later)
- `signal_extracted` (false — flipped to true after sieve pass processes it)
- `pass_number` (null — which sieve pass classified this email)

### Step 2.3 — Basic stats
Count and log:
- Total emails per mailbox
- Emails per year
- Top 50 sender domains
- Top 50 sender email addresses
- Self-sent count
- Emails with attachments count

### Step 2.4 — Generate first spreadsheet
Export `master-sheet.xlsx` with all emails sorted by date. Columns: date, year, from, to, subject, domain, is_self_sent, language, body_length. No classifications yet — just raw data you can scroll through.

**Output**: 
- `mailo.db` with every email as a row
- `master-sheet.xlsx` v1 (raw, sorted by date)
- Console log with basic stats

---

## PHASE 3: SIEVE — Layer by layer ($0 for passes 1-4, ~$0.50 for pass 5)

Each pass targets one content type. For each email it touches:
1. Classifies: is this email this content type? (yes/no)
2. Extracts: what signal can we pull from it?
3. Marks: updates the database row with `content_type`, `pass_number`, `signal_extracted`

After all sieve passes, every email in the database has a `content_type`. What's left unclassified after all passes = the real stuff.

---

### Pass 1 — Automated / Machine Emails ($0, pure regex)

**Detection rules** (no AI needed):
- `from_email` contains "noreply", "no-reply", "mailer-daemon", "notifications"
- `from_email` domain is known automated sender (github.com, vercel.com, google.com/accounts, etc.)
- Body contains "unsubscribe" link
- Body contains "This is an automated message"
- Body matches known service notification patterns

**Signal extraction** (write to `signal-log.json`):
```json
{
  "pass": 1,
  "type": "automated",
  "signals": [
    { "date": "2014-09-30", "signal": "Registered for Workaway.info", "source": "email-xxx" },
    { "date": "2016-10-15", "signal": "Started using Technion campus email", "source": "email-xxx" },
    { "date": "2022-01-01", "signal": "COVID Green Pass issued", "source": "email-xxx" },
    { "date": "2026-01-09", "signal": "TailorPlayed first Stripe order TP-2026-00001", "source": "email-xxx" }
  ]
}
```

**What we extract**: Service registration dates, account creation dates, platform usage periods, subscription starts/ends. These are timestamps for the life timeline.

**Database update**: Set `content_type = "automated"`, `action = "delete"`, `pass_number = 1`, `signal_extracted = true`.

**Expected volume**: ~30-40% of all emails.

---

### Pass 2 — Spam / Marketing ($0, regex + domain lists)

**Detection rules**:
- Known marketing domains (mailchimp, sendinblue, constantcontact, etc.)
- Subject patterns: "% off", "sale", "limited time", "unsubscribe"
- Bulk political emails (israelim.mail@mytnews.com, campaign senders)
- Repeated identical senders with promotional content

**Signal extraction**:
```json
{
  "pass": 2,
  "type": "marketing",
  "signals": [
    { "date": "2012-09", "signal": "On Bayit Yehudi/Naftali Bennett mailing list", "source": "email-xxx" },
    { "date": "2017-07", "signal": "Bought fake Facebook likes from Authentichits", "source": "email-xxx" },
    { "date": "2022-05", "signal": "Dave Matthews Band newsletter subscriber", "source": "email-xxx" }
  ]
}
```

**What we extract**: Political interests/periods, brand affinities, entertainment preferences, purchase patterns.

**Database update**: `content_type = "marketing"`, `action = "delete"`, `pass_number = 2`.

**Expected volume**: ~10-15% of all emails.

---

### Pass 3 — Newsletters / Mailing Lists ($0, pattern matching)

**Detection rules**:
- List-Unsubscribe header present
- Sender appears 5+ times with similar subject patterns
- Known newsletter senders (Nachshon mechina updates, Unit 81 alumni, etc.)
- Consistent periodic sending pattern (weekly, monthly)

**Signal extraction**:
```json
{
  "pass": 3,
  "type": "newsletter",
  "signals": [
    { "date_range": "2015-2018", "signal": "Active in Szarvas/JDC mailing list", "source": "multiple" },
    { "date_range": "2023-present", "signal": "Nachshon Mechina alumni - receives post-Oct 7 updates", "source": "email-xxx" },
    { "date_range": "2021-2022", "signal": "Unit 81 alumni association member", "source": "email-xxx" }
  ]
}
```

**What we extract**: Community memberships, organizational affiliations over time, periods of engagement/disengagement.

**Database update**: `content_type = "newsletter"`, `action = "delete"` (unless importance flag), `pass_number = 3`.

**Expected volume**: ~5-10%.

---

### Pass 4 — Transactional / Receipts / Logistics ($0, pattern matching)

**Detection rules**:
- From known transactional domains (booking.com, airbnb, elal.co.il, paybox, buyme, etc.)
- Subject contains: "confirmation", "receipt", "booking", "order", "invoice", "אישור", "הזמנה"
- Flight confirmations, hotel bookings, purchase receipts
- Campus print notifications (Technion)
- Medical appointments, insurance

**Signal extraction** (this layer is RICH):
```json
{
  "pass": 4,
  "type": "transactional",
  "signals": [
    { "date": "2009-10-11", "signal": "Flight Detroit→Amsterdam→Tel Aviv (Diller return)", "source": "email-xxx" },
    { "date": "2014-08-03", "signal": "Flight to Budapest (Szarvas 2014)", "source": "email-xxx" },
    { "date": "2017-08-20", "signal": "In Riga, Latvia (hostel booking)", "source": "email-xxx" },
    { "date": "2019-05", "signal": "Budapest trip with Gal (Butzipesht)", "source": "email-xxx" },
    { "date": "2019-10-22", "signal": "Warsaw trip with Gal (LOT Polish, Puro Hotel)", "source": "email-xxx" },
    { "date": "2020-03-23", "signal": "Filed for unemployment (COVID)", "source": "email-xxx" },
    { "date": "2025-10", "signal": "Barcelona trip (El Al, Airalo eSIM)", "source": "email-xxx" }
  ]
}
```

**What we extract**: Complete travel timeline, purchase history, addresses, phone numbers, medical providers, financial services. This is the skeleton of the life timeline.

**Database update**: `content_type = "transactional"`, `action = "delete"` (but signal preserved), `pass_number = 4`.
**Exception**: Some transactional emails are sentimental (Ben planning Vienna trip for mom+Natan). Flag these manually later.

**Expected volume**: ~10-15%.

---

### Pass 5 — Content Type Classification (~$0.50, light AI)

**What's left**: After passes 1-4, roughly 30-40% of emails remain. These are the real human communications. But they're still a mix of:
- Professional logistics (meeting scheduling, file sharing, work coordination)
- Group organizational emails (Szarvas planning, Diller coordination, Technion study groups)
- Personal correspondence (letters, emotional exchanges, friendship)
- Self-sent archives (Ben emailing himself files and notes)
- Creative work (designs, presentations, documents Ben created)

**Method**: Send each remaining email through a CHEAP LLM (GPT-4o-mini via batch API) with a simple classification prompt:

```
Classify this email into ONE category:
- professional_logistics (work scheduling, task coordination, file sharing)
- organizational (group planning for programs, camps, events)
- personal (emotional content, friendship, family, personal matters)
- self_archive (user emailing themselves files or notes)
- creative_work (designs, presentations, articles, creative content)
- study_materials (homework, lecture notes, academic exchanges)
- file_share (just a file being sent, no meaningful text)

Return JSON: { "category": "...", "confidence": 0.0-1.0, "one_line_summary": "..." }
```

**Database update**: Set `content_type` to the classified category, `pass_number = 5`.

**Signal extraction**: The `one_line_summary` gets stored. For self-archives and creative work, extract what was created and when.

**Output after Pass 5**: Every single email in the database now has a `content_type`. The spreadsheet shows the full picture.

---

### Pass 5b — Year + Era Labeling ($0, code only)

After all sieve passes, run automatic era detection:
- Every email already has a `year`
- Run PELT change-point detection on monthly email volume + contact diversity
- Auto-label detected eras: "2007-2008", "2009", "2009-2010", etc.
- Cross-reference with known life events from signal-log (travel dates, job changes, service registrations)
- Update `era` column for every email

**Output**: Each email now has both a `content_type` AND an `era`.

---

### Sieve Summary

| Pass | Target | Method | Cost | Emails caught | Signal extracted |
|------|--------|--------|------|---------------|-----------------|
| 1 | Automated/machine | Regex | $0 | ~30-40% | Service dates, platform usage |
| 2 | Spam/marketing | Regex + domains | $0 | ~10-15% | Interests, political affiliations |
| 3 | Newsletters | Pattern matching | $0 | ~5-10% | Community memberships, affiliations |
| 4 | Transactional | Pattern matching | $0 | ~10-15% | Travel timeline, purchases, addresses |
| 5 | Classify remainder | Cheap LLM | ~$0.50 | Remaining 30-40% | Content type + one-line summary |
| 5b | Era labeling | PELT algorithm | $0 | All emails | Era assignment |

**After all passes**: 
- Every email has: `content_type`, `pass_number`, `era`, `action` recommendation
- `signal-log.json` contains all extracted intelligence
- `master-sheet.xlsx` is regenerated with all classifications
- ~500-800 emails are tagged as `personal`, `creative_work`, or high-importance `organizational`

---

## PHASE 4: DEEP ANALYSIS (batches of 100-200, this is where Mailo comes alive)

**This phase doesn't happen in Cowork.** This is the conversation — you and Mailo (Claude), era by era or batch by batch.

### Step 4.1 — Pick a batch
Choose 100-200 emails. Options:
- By era: "Show me everything from 2009-2010 (Mechina Nachshon)"
- By person: "Show me all 47 emails involving Maya Wolk"
- By type: "Show me all emails classified as personal"
- By importance: "Show me the top 200 emails by importance score"

### Step 4.2 — Mailo analyzes the batch
For each email in the batch:
- Entity extraction (who is mentioned, what relationships are implied)
- Relationship classification (what is this person to Ben?)
- Emotional tone analysis
- Gem detection (jokes, meaningful quotes, beautiful writing)
- Cross-referencing with signal-log (does this email connect to a known event?)

### Step 4.3 — Mailo asks you questions
"I see 12 emails from Yael Kalif in this batch. The tone shifts dramatically in November 2017 — she writes a letter that starts with 'אהבה'. This seems connected to a major life event. Can you tell me what happened?"

### Step 4.4 — Build the network
Each analyzed batch adds to:
- **People directory** (new people, updated relationships, role evolution)
- **Timeline** (events confirmed, eras refined)
- **Insights** (patterns detected, connections discovered)
- **Gems collection** (preserved meaningful fragments)

### Step 4.5 — Update the database
Everything learned flows back into `mailo.db`:
- `importance` scores updated
- `people_ids` linked
- `action` confirmed (keep/delete)
- `label` finalized

---

## PHASE 5: EXECUTE (Optional, whenever you're ready)

### Step 5.1 — Review the spreadsheet
Open `master-sheet.xlsx`. Filter by `action = "delete"`. Scan for anything that shouldn't be deleted. Adjust.

### Step 5.2 — Run the Gmail script
A Python script reads the spreadsheet and:
- Applies Gmail labels to all "keep" emails according to the `label` column
- Moves all "delete" emails to trash (or applies `_TO_DELETE` label for manual bulk delete)
- Reports what it did

### Step 5.3 — Done
Your Gmail is clean. Your life record is built. Your people directory is complete.

---

## How Pieces Connect

The SQLite database is the spine. Everything references `email_id`:

```
email_id → content_type (from sieve passes)
email_id → era (from auto-detection)  
email_id → people_ids (from deep analysis)
email_id → signal extracted (in signal-log.json)
email_id → gem (in gems.json)
email_id → action + label (final classification)
```

The signal-log, gems, and people directory are SEPARATE files that reference back to email_ids. This means:
- You can always trace an insight back to its source email
- You can regenerate the spreadsheet at any point
- You can run additional analysis passes without losing previous work
- The database only grows — nothing is destroyed until you explicitly execute deletions on Gmail

---

## What You Need To Do

1. Export both mailboxes from Google Takeout (10 min)
2. Give Cowork the processing instructions + MBOX files
3. Wait for Phases 2-3 to complete (~30 min processing)
4. Review the spreadsheet
5. Open a new chat for Phase 4 (the conversation part)

That's it. Five steps.
