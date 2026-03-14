# Mailo — Complete Handoff Document
> Created: March 13, 2026
> Purpose: Everything needed to build the Mailo Cowork plugin from scratch
> Context: This document captures the full output of an intensive research and architecture session. A new Claude instance should be able to read this and build the plugin.

---

## PART 1: WHAT IS MAILO

Mailo is a **biographical intelligence engine** that processes personal data (email archives, WhatsApp exports, screenshots, Google Timeline, photos, documents) and extracts layered behavioral insights about the person's life, relationships, and patterns.

### What Mailo is NOT:
- Not a CRM (those manage relationships going forward)
- Not a memoir tool (those do voice interviews)
- Not a genealogy tool (those map family trees)
- Not an email organizer (those sort inbox)

### What Mailo IS:
A pipeline that takes ANY personal data input, attacks it from multiple angles at multiple depth levels, extracts every layer of intelligence, stores the intelligence in structured form, and discards the source material. The user keeps a clean inbox/phone. The life record lives in Mailo's intelligence database.

### The core insight:
**Statistics find the signal. LLMs tell the story.** 70-80% of insights come from metadata alone (who emailed whom, when, how often). The LLM only comes in for the 20-30% that requires understanding language, tone, and implicit meaning.

### The 9 depth levels (Mailo's unique IP):

**Surface Tier (Levels 1-3): What happened?**
1. **Basic** — "This email is from Maya, dated June 2010, about camp supplies"
2. **Surface** — "Maya is a coordinator who sends camp logistics"
3. **Understandable** — "She's been emailing you since 2007 across two programs"

**Analytical Tier (Levels 4-6): What does it mean?**
4. **Continuous** — "Appears in 9 periods. Tone shifts formal→warm"
5. **Deep** — "Gave you increasing autonomy year over year"
6. **Indirect** — "IDF letter = institutional advocacy at personal cost"

**Profound Tier (Levels 7-9): What deep truth does this reveal?**
7. **Very Deep** — "Functions as professional identity mother-figure"
8. **Surprising** — "Her family connects to another key person you didn't realize"
9. **Recurring** — "You seek mentor figures who give autonomy — same pattern across 3 life domains"

---

## PART 2: THE ARCHITECTURE

### The Sieve Model

Mailo does NOT classify everything at once. It works in layers — extracting value then removing, each pass making the pile smaller and the remaining pile more valuable.

**Pass 1 — Automated/machine emails** ($0, regex):
- Detection: noreply@, known automated domains, "unsubscribe" links
- Signal extracted: Service registration dates, platform usage periods, subscription starts/ends
- Action: Mark for deletion, preserve signals
- Expected: ~30-40% of emails

**Pass 2 — Spam/marketing** ($0, regex + domain lists):
- Detection: Marketing domains, promotional subject patterns, political bulk mail
- Signal extracted: Interests, political affiliations, brand preferences
- Action: Mark for deletion, preserve signals
- Expected: ~10-15%

**Pass 3 — Newsletters/mailing lists** ($0, pattern matching):
- Detection: List-Unsubscribe header, repeated sender patterns, periodic sending
- Signal extracted: Community memberships, organizational affiliations, engagement periods
- Action: Mark for deletion, preserve signals
- Expected: ~5-10%

**Pass 4 — Transactional/receipts** ($0, pattern matching):
- Detection: Known transactional domains, confirmation/receipt subject patterns
- Signal extracted: Travel timeline, purchases, addresses, phone numbers, medical providers
- Action: Mark for deletion, preserve signals (RICH layer — skeleton of life timeline)
- Expected: ~10-15%

**Pass 5 — Content type classification** (~$0.50, cheap LLM):
- Target: Remaining ~30-40% (the real human communications)
- Classification into: professional_logistics, organizational, personal, self_archive, creative_work, study_materials, file_share
- Also extracts: one-line summary per email
- Action: Classify, prepare for deep analysis

**Pass 5b — Era labeling** ($0, PELT algorithm):
- Automatic life-era detection from email patterns
- Change-point detection on monthly volume + contact diversity
- Each email gets an era label

### After sieve: Every email has a content_type and an era. ~500-800 emails are flagged as valuable for deep analysis.

### Deep Analysis (Phase 4):
- Works in batches of 100-200 emails
- This is where the conversation happens (human in the loop)
- Entity extraction, relationship classification, emotional tone, gem detection
- Builds: people directory, timeline, insights, gems collection

---

## PART 3: THE PLUGIN ARCHITECTURE

Mailo is a Cowork plugin with multiple skills. **Skills are activated by Mailo itself, NOT by the user.** The user feeds data and answers questions. Mailo routes internally.

### Decision logic:
When Mailo receives input, it identifies the type and activates the appropriate skill chain automatically. For example:
- User drops MBOX file → Ingest skill (identifies as email archive) → Sieve skill (runs passes 1-5) → Timeline skill (era detection) → reports back
- User drops screenshot → Ingest skill (identifies as image) → extracts metadata, text, behavioral signals, emotional reading, relational dynamics, generates gap questions
- User asks "tell me about Maya" → People skill (pulls everything known about Maya across all sources)
- User says "what questions do you have for me?" → Gap skill (surfaces open questions)

### Plugin structure:

```
mailo-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── ingest/SKILL.md       ← Receives new data, identifies type, routes to correct processing
│   ├── sieve/SKILL.md        ← Layered extraction passes (1-5b) for email archives
│   ├── people/SKILL.md       ← People directory: merge identities, build profiles, track evolution
│   ├── timeline/SKILL.md     ← Life timeline: detect eras, place events, connect dots
│   ├── gap/SKILL.md          ← Open questions: surface to user, check new data against them
│   ├── insight/SKILL.md      ← Deep analysis: patterns, archetypes, hidden connections (Levels 5-9)
│   └── execute/SKILL.md      ← Gmail operations: apply labels, delete emails
├── connectors/                ← Gmail connector
└── state/                     ← Mailo's persistent state (the intelligence database)
    ├── mailo.db               ← SQLite: all email metadata + classifications
    ├── signal-log.json        ← Intelligence extracted from sieve passes
    ├── people.json            ← People directory with evolving profiles
    ├── timeline.json          ← Eras + events
    ├── gems.json              ← Preserved meaningful fragments
    ├── gaps.json              ← Open questions waiting for answers
    └── insights.json          ← Deep patterns and connections
```

### Skill descriptions (for SKILL.md files):

**Ingest**: "When the user provides any new data source (file, image, text, export), identify the data type and route to the appropriate processing pipeline. Supported types: MBOX email archive, Gmail Takeout export, WhatsApp chat export, Google Timeline/Maps export, screenshot image, PDF document, text input. For each type, extract maximum intelligence at all available depth levels before discarding source material."

**Sieve**: "Process email archives through 5 sequential extraction passes, each targeting a specific content type. Extract signal from each layer before marking it for removal. Passes run in order: automated/machine → spam/marketing → newsletters → transactional → content classification. After all passes, run PELT change-point detection for era labeling. Update mailo.db with all classifications."

**People**: "Manage the people directory. When a new person is encountered, check against existing entries using email matching, name fuzzy matching (Jaro-Winkler), and Hebrew-English cross-lingual matching. Merge duplicates. For each person, track: all known names/aliases, email addresses, relationship to user (with temporal evolution), roles, era appearances, connection to other people, importance score. Generate and store open questions about ambiguous people."

**Timeline**: "Manage the life timeline. Detect eras automatically from email patterns (PELT on monthly multivariate time series). Place events from signal-log and deep analysis. Connect events to people and eras. Maintain chronological consistency. When new data arrives, check if it fills timeline gaps."

**Gap**: "Manage open questions. When any skill encounters missing information (who is this person? why did this relationship change? where was the user living at this time?), generate a specific question and store it. When new data arrives, check against open questions. When the user activates gap-filling mode, present questions one at a time, prioritized by how many connections the answer would unlock."

**Insight**: "Run deep analysis on processed data. Levels 5-9: behavioral inference (coding directive vs. consultative language over time), implicit meaning extraction (what does this action reveal?), archetypal role identification (comparing across all mentor/authority figures), hidden network connections (shortest-path analysis between seemingly unrelated people), cross-domain pattern synthesis (same relational pattern across camp/university/work)."

**Execute**: "When the user is ready, apply classifications back to Gmail. Read mailo.db action/label columns, apply Gmail labels to 'keep' emails, move 'delete' emails to trash or apply _TO_DELETE label. Always ask for confirmation before executing. Report what was done."

---

## PART 4: MULTI-ANGLE EXTRACTION (THE MAGIC)

When Mailo receives ANY piece of data, it doesn't just classify it. It extracts from every possible angle:

### For an email:
- **Metadata**: Date, time, sender, recipient, thread, CC, domain
- **Content surface**: Topic, entities mentioned, one-line summary
- **Behavioral signals**: Time of day (night owl?), response speed, formality level, language choice
- **Emotional reading**: Tone, sentiment, warmth vs. distance
- **Relational signals**: How does sender address user? Greeting style evolution. Power dynamics.
- **What's NOT there**: Expected response that didn't come? Someone missing from CC who should be there?

### For a screenshot of a conversation:
- **Metadata**: Date, time, platform, device
- **Content**: What's being discussed?
- **Speech patterns**: Both parties — slang, formality, emoji, length
- **Emotional reading**: Who's happy? Who's frustrated? Who's asking vs. telling?
- **Relational dynamics**: Power balance, trust level, openness
- **Background signals**: Screen wallpaper, notification bar, battery level (stressed = low battery at 2am?)
- **What's NOT there**: Who's missing? What's not being said? Generate gap questions.

### For Google Timeline/Maps data:
- **Locations visited**: Home, work, travel destinations
- **Patterns**: Daily routine, weekend habits, travel frequency
- **Transitions**: When did commute change (job change?), new regular location (new relationship?)
- **Cross-reference**: Match locations to email correspondence (emails from Budapest = Szarvas period)

### For WhatsApp export:
- **Same as email** but with higher frequency data
- **Response time patterns**: Who responds instantly vs. hours later?
- **Media sharing patterns**: Photos, voice messages, links
- **Group dynamics**: Who talks most, who reacts, who stays silent

---

## PART 5: THE STATE MODEL

Mailo lives in a folder and has persistent state. When activated, it reads its state and knows:
- How many sources have been processed
- How many people are identified
- How many eras are detected
- How many open questions exist
- What the user's last interaction was about
- What processing is pending

### mailo.db (SQLite) — core email table columns:
```
email_id, mailbox, date, year, month, from_email, from_name, to_emails, cc_emails,
subject, body_text, body_length, has_attachment, attachment_names, thread_id,
is_self_sent, language, domain,
content_type, importance, action, label, people_ids, era, signal_extracted, pass_number
```

### signal-log.json — extracted intelligence from sieve passes:
```json
{
  "signals": [
    {
      "date": "2014-09-30",
      "signal": "Registered for Workaway.info",
      "source_email_id": "xxx",
      "pass": 4,
      "type": "service_registration",
      "tags": ["travel", "volunteering"]
    }
  ]
}
```

### people.json — evolving people directory:
```json
{
  "people": [
    {
      "id": "person-001",
      "name_display": "Maya Wolk",
      "name_original": "מאיה וולק",
      "emails": ["wolk.maya@gmail.com", "mayawo@netvision.net.il"],
      "importance": 5,
      "first_seen": "2007",
      "last_seen": "2018",
      "roles": [
        { "role": "coordinator", "context": "Diller", "period": "2007-2009" },
        { "role": "mentor", "context": "Szarvas", "period": "2010-2018" }
      ],
      "relationship_evolution": [
        { "period": "2007-2009", "nature": "authority/coordinator" },
        { "period": "2010-2014", "nature": "mentor giving autonomy" },
        { "period": "2015-2018", "nature": "trusted peer/friend" }
      ],
      "era_appearances": ["era-01", "era-04", "era-06", "era-10a", "era-12"],
      "connections": [
        { "person_id": "person-042", "type": "family_link", "note": "Husband is Yonatan's cousin" }
      ],
      "status": "confirmed"
    }
  ]
}
```

### timeline.json:
```json
{
  "eras": [
    {
      "id": "era-01",
      "name": "Havayah 2008 + Genesis",
      "date_from": "2007-10",
      "date_to": "2008-06",
      "auto_detected": true,
      "confirmed": true,
      "people_ids": ["person-001", "person-015"],
      "summary": "Entry into Jewish youth programming. First creative recognition."
    }
  ],
  "events": [
    {
      "id": "event-001",
      "type": "milestone",
      "name": "First published article (JVibe)",
      "date": "2009-08",
      "era_id": "era-04",
      "people_ids": ["person-055"],
      "significance": 5
    }
  ]
}
```

### gems.json:
```json
{
  "gems": [
    {
      "id": "gem-001",
      "type": "quote",
      "content": "חיכית חיכית, בכית בכית, ומי לא בא?",
      "context": "Maya Wolk's school letter for Ben's Genesis application",
      "person_id": "person-001",
      "source_email_id": "xxx",
      "date": "2008",
      "tags": ["humor", "mentorship"],
      "user_note": null
    }
  ]
}
```

### gaps.json:
```json
{
  "gaps": [
    {
      "id": "gap-001",
      "question": "Who is Ahud whose mother died in June 2012? Maya said 'she was my teacher.'",
      "generated_from": "email-xxx",
      "date_generated": "2026-03-15",
      "priority": "medium",
      "connections_it_would_unlock": ["person-001", "era-06"],
      "status": "open",
      "answer": null
    }
  ]
}
```

---

## PART 6: RESEARCH FINDINGS (KEY TAKEAWAYS)

Three deep research tracks were conducted. Here are the findings that matter for implementation:

### From Research 1 (Behavioral Signal Extraction):
- Netflix, Spotify, Amazon extract personality from implicit signals (what you DO, not what you SAY)
- Hu-Koren-Volinsky confidence weighting: `c = 1 + α × r` where interaction count scales confidence
- Per-contact engagement score (response speed × reply length × thread depth × forwarding frequency) reveals TRUE relationship hierarchy
- Session-based patterns: morning email sequence reveals cognitive priority hierarchy
- Koren's TimeSVD++: temporal dynamics > model complexity. The 2012 user ≠ the 2025 user.

### From Research 2 (Personal Data Pipeline):
- **Entity resolution**: Splink library (pip install, Fellegi-Sunter on DuckDB, 1M records/minute, no training data needed). Use Jaro-Winkler for fuzzy name matching (F1 ~0.89). Hebrew-English name pairs need lookup table + LLM fallback.
- **Relationship signals**: Intimacy contributes 32.8% to tie-strength vs. frequency at 19.7% (Gilbert & Karahalios). Greeting/closing style is the single most reliable content indicator.
- **Hebrew NLP**: Frontier LLMs handle mixed Hebrew-English natively. Hebrew costs ~4x more tokens. DictaBERT-seg for morphological precision. HeBERT for sentiment (F1 ~0.96).
- **Era detection**: PELT algorithm (ruptures library, 5 lines of Python). Feed monthly email volume + contact diversity. Detects 5-15 change points that align with life transitions.
- **Cost**: 5,000 emails fully processed for $4-$8 using tiered architecture.

### From Research 3 (Progressive Context Layering):
- Intelligence analysis techniques (CIA SATs, ACH, pattern-of-life) map to email analysis
- ICIJ processed 11.5M files for Panama Papers using Apache Tika + Neo4j + NER
- GraphRAG (Microsoft 2024): LLM-generated knowledge graphs integrated into RAG
- RAPTOR trees (Stanford): cluster → summarize → cluster summaries → produces hierarchy from emails to life themes
- Key: every insight must trace back to specific source emails (provenance = trust)
- Graph techniques: Leiden community detection for social circles, betweenness centrality for bridge people, shortest-path for hidden connections

### Critical cost finding:
- Passes 1-4 (sieve): $0 (pure regex/pattern matching)
- Pass 5 (cheap LLM classification): ~$0.50 for 5,000 emails via batch API
- Deep analysis (Levels 5-9): ~$2-5 total, only on ~500 high-value emails
- Total: **$4-$8 for a complete 5,000-email archive**

---

## PART 7: BEN'S CONTEXT (FOR THE BIOGRAPHICAL ANALYSIS)

Ben Akiva is the first user. His archive:
- **dartaryan@gmail.com**: ~3,686 emails, 2007-2026 (primary, already triaged in batches 1-31+)
- **benakiva1991@gmail.com**: secondary, not yet processed
- **Google Takeout**: Both mailboxes exported as MBOX + Google Maps/Timeline data

### Key facts about Ben (relevant for analysis):
- Israeli, born ~January 24, 1991, based in Tel Aviv
- Gay (came out November 2017 — structural pivot in the timeline)
- Partner: Gal (together since early 2019)
- Sister: Mey (b. May 30, 1988), works at NVIDIA
- Mother: Ofira (artist — actress, theater director, painter)
- Father: Itzik (Yitzchak) Akiva; parents divorced when Ben was in 4th grade
- Stepfather: Natan Ben-dov (deceased) — deeply loved
- Grandmother Nediva (deceased ~November 2010) — "the person I loved most in the world"

### Key life threads:
- **Szarvas International Jewish Youth Camp** (JDC-Lauder, Hungary): appears in nearly every era of adult life
- **Builder-Documenter personality**: builds things for others and documents compulsively
- **Maya Wolk**: central mentor figure across all delegation programs and Szarvas
- **Ehud Kuper ("Budiru")**: friendship from age 6 through present
- **Or Karni**: INTJ to Ben's ENFJ, trusted creative critic
- **Yael Kalif**: deep friendship, wrote "אהבה" letter when Ben came out

### Existing processed eras (from previous Mailo sessions):
Eras 01-10b have been processed chronologically with narrative files, email registries, and people directory entries. This existing data should be ingested as another data source — not discarded.

### Existing files location:
```
C:\Users\darta\Desktop\פרויקטים\gmail-llm-labeler\.claude\skills\triage\life-story-agent\mailo\
├── progress.md
├── people-directory.md (~30KB)
├── label-taxonomy.md
├── life-record/ (one file per completed era)
├── email-registry/ (one file per completed era)
└── task-files/
```

---

## PART 8: DESIGN PRINCIPLES

1. **Mailo decides which skills to activate** — the user never manually selects a skill. The user feeds data, answers questions, confirms deletes. Mailo routes internally.

2. **Extract then discard** — Mailo doesn't hoard source material. It extracts intelligence and lets the user delete the originals. Clean inbox, rich life record.

3. **Multi-angle attack** — every piece of data gets analyzed from metadata, content, behavioral, emotional, relational, and "what's missing" angles simultaneously.

4. **Gap questions are first-class citizens** — when Mailo encounters missing information, it generates specific questions and stores them. These either get answered by future data automatically, or surface to the user in gap-filling mode.

5. **User annotations add meaning** — the user can say "this was my last conversation with him" or "this made me laugh." These become part of the intelligence. But Mailo works without them.

6. **Ask before deleting** — Mailo recommends deletions but always asks for confirmation before executing on real Gmail.

7. **Everything connects through IDs** — email_id, person_id, era_id, event_id. Every insight traces back to sources. Provenance = trust.

8. **Hebrew preserved** — original Hebrew names, subjects, quotes are preserved as-is alongside English descriptions.

---

## PART 9: FIRST SESSION PLAN

When the new Cowork session starts:

1. **Build the plugin structure** — create the folder structure, plugin.json, all SKILL.md files
2. **Set up the state folder** — create empty JSON files and SQLite schema
3. **Point at the MBOX files** — Ben has already exported from Google Takeout
4. **Run Phase 2 (Parse)** — extract all emails into the database
5. **Run Phase 3 (Sieve passes 1-4)** — regex-based extraction and classification
6. **Run Phase 3 Pass 5** — cheap LLM classification of remaining emails
7. **Run Phase 3 Pass 5b** — PELT era detection
8. **Generate master spreadsheet** — the reviewable output
9. **Ingest existing Mailo data** — the 10+ completed eras become input
10. **Report to Ben** — "I've processed X emails, found Y people, detected Z eras, have Q open questions"

---

## PART 10: TOOLS AND LIBRARIES

All pip-installable:

| Function | Tool | Why |
|----------|------|-----|
| Email parsing | Python `email` + `mailbox` stdlib | Handles MBOX natively |
| Quoted text removal | talon (Mailgun) | SVM-based signature/quote stripping |
| Entity resolution | splink | Fellegi-Sunter, 1M records/min, no training data |
| Name matching | jellyfish (Jaro-Winkler) | F1 ~0.89 on names |
| Contact graph | networkx | Community detection, centrality, shortest paths |
| Change-point detection | ruptures | PELT algorithm, 5 lines of code |
| Topic modeling | bertopic | Dynamic topics over time |
| Sentiment (Hebrew) | HeBERT via transformers | F1 ~0.96 |
| Embeddings | sentence-transformers | Free, local, multilingual |
| Anomaly detection | scikit-learn (IsolationForest) | 15 lines |
| Database | sqlite3 (stdlib) | Zero config, single file |
| LLM calls | anthropic (batch API) | 50% discount, prompt caching |

---

## APPENDIX: FILES PRODUCED IN THIS SESSION

Three files were created during this research session:

1. **mailo-schema-proposal.md** — Initial schema design (superseded by the sieve architecture but contains useful JSON examples for people, timeline, emails, creative works, insights, places)

2. **mailo-unified-architecture.md** — Synthesized architecture from 3 research tracks. Contains the 5-tier pipeline, 9-depth-level framework, tool recommendations, cost estimates, and build timeline.

3. **mailo-pipeline-steps.md** — Detailed step-by-step processing plan with sub-steps and data flow. Contains the sieve pass definitions and Phase 4 deep analysis workflow.

All three are in the outputs and should be read alongside this handoff document.
