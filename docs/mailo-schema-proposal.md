# Mailo Schema Proposal — Biographical Intelligence Data Model
> Research Date: March 13, 2026
> Purpose: Define the data architecture for Mailo as a scalable product

---

## Part 1: Research Findings

### What Exists Today (and Why None of It Is Mailo)

**Personal CRMs** (Clay, Dex, Monica, Nimble, Folk)
- Focus: Managing relationships going *forward* — follow-up reminders, networking
- Data model: Contact → Interactions → Tags → Reminders
- Gap: No retrospective analysis. No biographical narrative. No "life eras." No psychological pattern extraction. No email archive ingestion as primary input.

**AI Memoir/Biography Tools** (Life Story AI, Autobiographer, Storii, StoriedLife)
- Focus: Voice-based guided interviews → printed book
- Data model: Questions → Recordings → Transcripts → Chapters
- Gap: Entirely interview-driven, not artifact-driven. You tell the AI your story. Mailo reads your story FROM your data. These tools can't ingest an email archive. No structured people directory, no timeline, no cross-referencing.

**Genealogy Tools** (GEDCOM, FamilySearch, Gramps)
- Focus: Family trees — birth, marriage, death, parentage
- Data model: Person → Family → Event (birth/marriage/death) → Source
- Gap: Too narrow. Only models family relationships. No support for friendships, mentors, colleagues, camp counselors, army commanders. No creative work tracking. No psychological insights. No "eras."

**Knowledge Graphs / Temporal KGs** (Neo4j, RDF, academic TKGs)
- Focus: Academic — entity-relationship modeling with temporal dimensions
- Data model: (Subject, Predicate, Object, Timestamp) quadruples
- Useful concept: **Temporal scoping** — relationships have validity periods. Maya Wolk is "coordinator" in 2008 but "mentor" by 2014.
- Gap: Requires graph database infrastructure. Too complex for a file-based product. No predefined biographical ontology.

**Monica CRM** (closest spiritual match)
- Open source PRM (Personal Relationship Management)
- Tracks: contacts, interactions, notes, reminders, life events, gifts, debts
- Vision evolved from "personal CRM" to "document your life"
- Gap: Manual data entry only. No email ingestion. No narrative/era structure. No pattern analysis.

### The Gap Mailo Fills

**Mailo is retroactive biographical archaeology from digital artifacts.**

Nobody does this. The workflow is:
1. Ingest email archive (and eventually WhatsApp, photos, documents)
2. Automatically map people, events, places, creative works onto a timeline
3. Build evolving relationship profiles (who was this person to you, and when?)
4. Surface patterns, connections, insights
5. Enable the user to enrich, correct, and explore their own life story
6. Produce tangible outputs: letters, narratives, people profiles, timelines

---

## Part 2: Design Principles

1. **JSON, not graph database** — Must work as portable files. No server required. Readable by humans and LLMs. Compatible with Cowork, Claude Projects, local storage.

2. **Flat collections with cross-references** — Instead of deeply nested objects, use flat arrays of typed entities connected by IDs. This allows: querying ("show me all people from Szarvas"), filtering ("events in 2014"), and independent updates (add a person without touching the timeline).

3. **Temporal scoping everywhere** — Every relationship, role, and attribute can have `active_from` and `active_until`. Maya Wolk can be tagged as `coordinator → mentor → peer` across different periods.

4. **Source attribution** — Every fact links back to one or more source emails (or WhatsApp messages, or user statements). This is what makes Mailo trustworthy.

5. **Verification status** — Facts are either `inferred` (from automated triage), `confirmed` (validated by user), or `corrected` (user overrode the triage). This preserves the user-as-final-authority principle.

6. **Progressive enrichment** — The schema starts sparse and fills in over time. A person entry might begin with just a name and email, then gain roles, relationships, eras, and narrative significance as more data is processed.

7. **Importance scoring** — Not everything matters equally. A 1-5 scale on people, events, and emails lets the system surface what's significant.

8. **Bilingual support** — Original Hebrew (or any language) is preserved alongside English translations/descriptions. `name_original` + `name_display`.

---

## Part 3: The Schema

### Overview — Six Core Collections

```
mailo-data/
├── meta.json              # Project metadata, owner, settings
├── people.json            # All people in the life record
├── timeline.json          # Eras + events on a unified timeline
├── emails.json            # Email registry with classifications
├── creative-works.json    # Things the user created
├── insights.json          # Analytical observations and patterns
└── places.json            # Significant locations
```

### 3.1 meta.json — Project Metadata

```json
{
  "schema_version": "1.0.0",
  "project_name": "Mailo Life Record",
  "owner": {
    "name": "Ben Akiva",
    "birth_date": "1991-01-24",
    "primary_email": "dartaryan@gmail.com",
    "secondary_emails": ["benakiva1991@gmail.com"],
    "nationality": "Israeli",
    "languages": ["Hebrew", "English"]
  },
  "sources": [
    {
      "id": "src-gmail-dartaryan",
      "type": "gmail",
      "email": "dartaryan@gmail.com",
      "total_emails": 3686,
      "date_range": { "from": "2007-07", "to": "2026-03" },
      "status": "processing"
    }
  ],
  "processing_log": [
    {
      "date": "2025-09-15",
      "action": "initial_triage",
      "details": "41 batches processed by classification agent"
    }
  ],
  "settings": {
    "language_primary": "he",
    "language_output": "en",
    "importance_threshold_keep": 3,
    "auto_delete_spam": true
  }
}
```

### 3.2 people.json — People Directory

Each person is an entity with temporal roles and cross-era presence.

```json
{
  "people": [
    {
      "id": "person-001",
      "name_display": "Maya Wolk",
      "name_original": "מאיה וולק",
      "nicknames": [],
      "emails": ["wolk.maya@gmail.com", "mayawo@netvision.net.il"],
      "phone": null,
      "importance": 5,
      "first_appearance": "2007",
      "last_appearance": "2018",
      "status": "confirmed",

      "roles": [
        {
          "role": "coordinator",
          "context": "Diller program",
          "active_from": "2007",
          "active_until": "2009",
          "org_id": "org-diller"
        },
        {
          "role": "mentor",
          "context": "Szarvas camp",
          "active_from": "2010",
          "active_until": "2018",
          "org_id": "org-szarvas"
        }
      ],

      "relationship_to_owner": {
        "type": "mentor",
        "depth": "deep",
        "description": "Central mentor figure across all delegation programs and Szarvas. Gave Ben increasing autonomy. Parallel personality to Shalhevet Vardi.",
        "evolution": [
          { "period": "2007-2009", "nature": "authority/coordinator" },
          { "period": "2010-2014", "nature": "mentor giving autonomy" },
          { "period": "2015-2018", "nature": "trusted peer/friend" }
        ]
      },

      "era_appearances": ["era-01", "era-04", "era-06", "era-08", "era-10a", "era-12", "era-13", "era-16", "era-18"],

      "connections": [
        {
          "person_id": "person-042",
          "relationship": "husband is cousin of",
          "note": "Husband Yonatan is cousin of Yael"
        }
      ],

      "quotes": [
        {
          "text": "חיכית חיכית, בכית בכית, ומי לא בא?",
          "context": "School letter for Ben's Genesis application",
          "date": "2008",
          "source_email_id": null
        }
      ],

      "letter_status": "planned",
      "notes": "Still in contact. One of most important figures across Ben's adult life.",
      "last_updated": "2026-02-15"
    }
  ]
}
```

### 3.3 timeline.json — Eras & Events

The timeline has two levels: **Eras** (named life periods) and **Events** (specific moments within or across eras).

```json
{
  "eras": [
    {
      "id": "era-01",
      "name": "Havayah 2008 + Genesis Preparation",
      "slug": "havayah-2008",
      "date_from": "2007-10",
      "date_to": "2008-06",
      "location_ids": ["place-haifa", "place-brandeis"],
      "org_ids": ["org-havayah", "org-genesis"],
      "summary": "Ben's entry into Jewish youth programming. Designing at 2:30am for Joan Orkin. First creative recognition.",
      "themes": ["creative-awakening", "youth-programs", "first-recognition"],
      "people_ids": ["person-001", "person-015", "person-022"],
      "email_count": { "keep": 18, "delete": 24, "unsure": 3 },
      "processing_status": "completed",
      "processed_date": "2025-10-01",
      "narrative_file": "life-record/era-01-havayah-2008.md"
    }
  ],

  "events": [
    {
      "id": "event-001",
      "type": "milestone",
      "name": "First published article (JVibe)",
      "date": "2009-08",
      "date_precision": "month",
      "era_id": "era-04",
      "description": "Kali Foxman invited Ben to publish his Genesis experience. First magazine article at age 18.",
      "significance": 5,
      "people_ids": ["person-055"],
      "place_id": null,
      "source_email_ids": ["email-0342"],
      "tags": ["first", "writing", "identity"],
      "status": "confirmed"
    },
    {
      "id": "event-002",
      "type": "trip",
      "name": "Diller Congress — Ayia Napa, Cyprus",
      "date_from": "2009-07-15",
      "date_to": "2009-07-20",
      "era_id": "era-04",
      "description": "Israel Summer Seminar congress. Ben designed t-shirts, co-wrote closing speech with Shon Yaffe.",
      "significance": 4,
      "people_ids": ["person-078", "person-079"],
      "place_id": "place-ayia-napa",
      "source_email_ids": ["email-0310", "email-0315"],
      "tags": ["diller", "creative-work", "travel"],
      "status": "confirmed"
    },
    {
      "id": "event-003",
      "type": "loss",
      "name": "Grandmother Nediva passes away",
      "date": "2010-11",
      "date_precision": "month",
      "era_id": "era-06",
      "description": "The person Ben loved most in the world. Died during his first Szarvas summer.",
      "significance": 5,
      "people_ids": [],
      "place_id": null,
      "source_email_ids": [],
      "tags": ["loss", "family", "turning-point"],
      "status": "confirmed"
    }
  ],

  "event_types": [
    "milestone", "trip", "loss", "relationship_change", "career",
    "creative", "education", "identity", "family", "conflict",
    "celebration", "project", "meeting_someone"
  ]
}
```

### 3.4 emails.json — Email Registry

Replaces the current markdown email-registry files with structured, queryable data.

```json
{
  "source": "dartaryan@gmail.com",
  "emails": [
    {
      "id": "email-0342",
      "gmail_id": "18a3f2b1c4d5e6f7",
      "date": "2009-08-11",
      "from": "KFoxman@jflmedia.com",
      "to": ["dartaryan@gmail.com"],
      "cc": [],
      "subject": "JVibe article invitation",
      "era_id": "era-04",
      "classification": "keep",
      "label": "Life/Diller/2009/Memories",
      "importance": 5,
      "reason": "First published article invitation. Milestone.",
      "people_ids": ["person-055"],
      "event_ids": ["event-001"],
      "has_attachment": false,
      "attachment_names": [],
      "language": "en",
      "status": "confirmed",
      "label_applied": false,
      "gems": [
        {
          "type": "quote",
          "content": "We already have your headshot.",
          "note": "Implies Ben was already known to the publication"
        }
      ]
    }
  ]
}
```

**Key field: `gems`** — This is where jokes, meaningful quotes, surprising facts, and emotionally significant fragments are preserved. The triage agent can flag these automatically; the user confirms or adds more. This is what makes the archive *alive*, not just classified.

### 3.5 creative-works.json — Things Ben Created

```json
{
  "works": [
    {
      "id": "work-001",
      "title": "Diller 2009 Congress T-shirt Design",
      "type": "design",
      "date": "2009-07",
      "era_id": "era-04",
      "description": "T-shirt design for Cyprus congress. Yeshaya Tzavri served as printer liaison.",
      "collaborators": ["person-078"],
      "source_email_ids": ["email-0305", "email-0308"],
      "significance": 3,
      "status": "confirmed"
    },
    {
      "id": "work-002",
      "title": "Le Budiru — Ehud's Bedroom Renovation",
      "type": "physical_design",
      "date": "2013-01",
      "era_id": "era-07",
      "description": "Secret overnight renovation of Ehud Kuper's bedroom while he was away with the Paratroopers. Handmade cork board, design package.",
      "collaborators": ["person-003", "person-004"],
      "source_email_ids": ["email-1205"],
      "significance": 5,
      "status": "confirmed"
    }
  ],

  "work_types": [
    "design", "writing", "presentation", "letter", "physical_design",
    "digital_game", "board_game", "workshop", "video", "photography",
    "educational_activity", "shadow_puppet_show"
  ]
}
```

### 3.6 insights.json — Pattern Recognition & Analysis

This is what makes Mailo more than a filing system. These are the psychological patterns, behavioral observations, and cross-era connections that emerge from the data.

```json
{
  "insights": [
    {
      "id": "insight-001",
      "type": "behavioral_pattern",
      "title": "Builder-Documenter Personality",
      "description": "Ben doesn't just build — he documents while building. This pattern is visible from age 17 (Ori's birthday letter) through present (BMAD knowledge base). Most builders hate documentation. Most documenters don't build. Ben does both simultaneously.",
      "first_evidence_era": "era-02",
      "supporting_eras": ["era-04", "era-07", "era-10a", "era-12"],
      "supporting_events": ["event-001", "work-001", "work-002"],
      "confidence": "high",
      "source": "analyst",
      "status": "confirmed"
    },
    {
      "id": "insight-002",
      "type": "relationship_pattern",
      "title": "Maya Wolk as the Through-Line",
      "description": "Maya Wolk appears in more eras than any other person outside family. She functions as a mirror of Ben's growth — giving more autonomy as he matures. The Szarvas network, which touches almost every era of Ben's adult life, flows through her.",
      "supporting_people": ["person-001"],
      "supporting_eras": ["era-01", "era-04", "era-06", "era-08", "era-10a", "era-12", "era-13"],
      "confidence": "high",
      "source": "analyst",
      "status": "confirmed"
    },
    {
      "id": "insight-003",
      "type": "identity_evolution",
      "title": "Coming Out as Structural Pivot",
      "description": "November 2017 divides the timeline. Before: relationships are coded, identity is implicit. After: Tel Aviv, Gal, TailorPlayed, Zen Nadir — everything opens up. Yael Kalif's 'אהבה' letter is the archival marker of this transition.",
      "pivot_era": "era-17",
      "supporting_events": ["event-045"],
      "confidence": "high",
      "source": "analyst",
      "status": "confirmed"
    }
  ],

  "insight_types": [
    "behavioral_pattern", "relationship_pattern", "identity_evolution",
    "creative_growth", "coping_mechanism", "recurring_theme",
    "turning_point", "character_trait", "cross_era_connection"
  ]
}
```

### 3.7 places.json — Significant Locations

```json
{
  "places": [
    {
      "id": "place-szarvas",
      "name": "Szarvas International Jewish Youth Camp",
      "name_original": "סזרבש",
      "country": "Hungary",
      "type": "camp",
      "significance": 5,
      "era_ids": ["era-06", "era-08", "era-10a", "era-12", "era-13", "era-16", "era-18"],
      "description": "JDC-Lauder camp. The single most recurring location in Ben's life after Haifa and Tel Aviv."
    }
  ]
}
```

---

## Part 4: What This Schema Enables

### For Ben (completing his own archive)
- **Query**: "Who appears in more than 3 eras?" → Instant answer from `era_appearances` arrays
- **Query**: "Show me all events tagged 'first'" → All milestone firsts on one screen
- **Query**: "What did I create in 2014?" → Filter `creative-works.json` by date
- **Query**: "All emails from Maya Wolk, sorted by date" → Cross-reference `people_ids` in `emails.json`
- **Resume processing**: Next session starts by reading `meta.json` processing status, not a giant markdown file

### For Mailo as a Product
- **Automated ingestion**: LLM reads emails, outputs JSON matching this schema. No freeform markdown.
- **Progressive enrichment**: Start with sparse person entries (name + email). Each processing pass fills in roles, eras, relationships.
- **Smart questions**: Mailo reads the schema and asks: "I found 47 emails involving Maya Wolk across 9 time periods. She seems to be a mentor who gave you increasing responsibility. Is that right?" — because the structure enables this.
- **Visualization**: Timeline view, people network graph, era explorer — all trivial to build from structured JSON.
- **Export**: Generate markdown narratives, printable timelines, letter templates — all from the same structured source.
- **WhatsApp ingestion**: Same schema. WhatsApp messages become entries in `emails.json` (or a parallel `messages.json`) with the same cross-referencing.

---

## Part 5: Migration Path (Ben's Current Data → New Schema)

The existing Mailo data (10+ completed eras in markdown) can be migrated:

1. **people-directory.md → people.json**: Parse existing markdown table, create JSON entries. Add IDs. Cross-reference eras.
2. **life-record/era-XX.md → timeline.json**: Extract era boundaries, events, people references. Narrative text stays as separate files linked from the era entry.
3. **email-registry/era-XX.md → emails.json**: Parse markdown tables into structured entries.
4. **triage-knowledge.md** → Split across all collections (people, timeline, insights)
5. **progress.md → meta.json processing_log**

This migration can be done programmatically in one session.

---

## Part 6: Schema Design Decisions & Rationale

| Decision | Rationale |
|----------|-----------|
| JSON over graph DB | Portable, readable by LLMs and humans, works in files. No infrastructure. |
| Flat collections over nested | Enables independent updates. Add a person without touching the timeline. |
| IDs everywhere | Cross-referencing is the magic. "Show me everything about person-001 across all collections." |
| Temporal scoping on roles | The same person plays different roles at different times. This is biographical reality. |
| `gems` field on emails | The jokes, quotes, and surprises are what make this ALIVE, not just organized. |
| Separate insights collection | Analysis is a first-class citizen, not a footnote. This is what makes Mailo different from a filing system. |
| `status` field (inferred/confirmed/corrected) | Preserves the principle that the user's word is final while tracking what came from automation. |
| `importance` on everything | Not all things matter equally. Enables filtering and prioritization. |
| Narrative files as separate markdown | Long-form era narratives don't belong inside JSON. They're linked, not embedded. |

---

## Part 7: Open Questions for Ben

1. **Gems format** — Should we also preserve Hebrew email subjects as gems? Many of them are beautiful/funny.
2. **Organization entities** — Should Szarvas, Diller, Technion, etc. be full entities in their own collection (`orgs.json`), or are they adequately represented as attributes on other entities?
3. **Attachment tracking** — Some attachments are significant (Shaked Bar's farewell letter PDF, Natan's photos). Should we track attachment metadata more explicitly?
4. **Privacy levels** — If this becomes a product, should the schema have per-field privacy levels (e.g., "this person's email is private but their name and role are shareable")?
5. **Multiple users** — The current schema assumes one person's life. A product version would need multi-tenancy. Keep this simple for now?
