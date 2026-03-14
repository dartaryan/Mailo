# Mailo — Unified Architecture Blueprint
> Synthesized from 3 deep research tracks | March 13, 2026
> Purpose: The single document that defines how Mailo works

---

## The One-Line Summary

**Statistics find the signal. LLMs tell the story.**

This is the central architectural insight across all three research tracks. You don't need an LLM to detect that email volume dropped 70% in March 2018 — a 5-line algorithm does that. You don't need an LLM to know that Maya Wolk appears in 9 different time periods — a SQL query does that. You need the LLM to read the actual emails from that transition window and say: "This is when you left the army and went to Szarvas for the first time. Your communication style shifted from hierarchical to collaborative. And Maya was the bridge."

---

## What The Research Proved

### 1. Nobody is doing this
No commercial product performs automated life-era detection from email archives. No tool builds evolving relationship profiles from communication patterns. No product extracts behavioral personality insights and gives them back to the user. This is genuinely novel.

### 2. Metadata carries 70-80% of the signal — for free
Email headers alone (sender, recipient, date, CC, thread structure) reveal: relationship hierarchy, communication rhythms, social network topology, life phase transitions, organizational affiliations, and true priority vs. stated priority. All computable with pure Python. Zero API cost.

### 3. The cost is trivial
Processing 5,000 emails end-to-end: **$4-$8 total**. Processing 50,000 emails: **$50-$160**. The tiered architecture (cheap models for bulk, expensive for synthesis) makes this accessible to individual users.

### 4. Hebrew NLP crossed a threshold in 2024-2025
Frontier LLMs (Claude, GPT-4o) handle mixed Hebrew-English natively. Dedicated Hebrew models (DictaLM 3.0, NeoDictaBERT-bilingual) enable local processing. The only catch: Hebrew costs ~4x more in tokens than English due to tokenizer inefficiency.

### 5. The 9-depth-level model is Mailo's unique IP
No existing tool operates at levels 5-9 (behavioral inference, implicit meaning, archetypal roles, hidden network connections, cross-domain pattern synthesis). This is where Mailo becomes something genuinely new — not a filing system, not a CRM, but biographical intelligence.

---

## The Pipeline: 5 Tiers

```
RAW EMAIL ──→ [Tier 0: Parse] ──→ [Tier 1: Embed] ──→ [Tier 2: Classify] ──→ [Tier 3: Analyze] ──→ [Tier 4: Synthesize]
                 Code only          Embeddings          Cheap LLM              Smart LLM             Best LLM
                 $0.00              $0.04               $0.50                  $3.00                 $2.00
                 5,000 emails       5,000 emails        5,000 emails           500-2,000 emails      ~50 prompts
                 Levels 1-2         Clustering           Level 3               Levels 4-6            Levels 7-9
```

### Tier 0 — Structural Extraction (code only, $0.00)
- Parse all headers (From, To, CC, Date, Subject, Thread-ID)
- Build contact frequency matrix
- Gmail-specific normalizations (dot-insensitivity, plus-addressing)
- Remove quoted text from replies (saves 30-50% of tokens downstream)
- Detect automated messages via regex (noreply@, unsubscribe links)
- Build conversation threads
- Per-contact metadata: first/last date, message count, response time, time-of-day distribution
- Language detection (Hebrew Unicode range vs. Latin)
- Domain categorization (corporate, personal, educational)

**Output**: SQLite database, NetworkX contact graph, cleaned email bodies, per-contact feature vectors.
**This tier alone handles ~30% of all analytical insights.**

### Tier 1 — Embedding & Clustering ($0.04)
- Embed all emails into vector space (text-embedding-3-small or free local model)
- Cluster by semantic similarity (HDBSCAN)
- Flag statistical outliers as potential "gems"
- Per-contact embedding centroids (what does each person typically discuss?)
- Enable semantic search across entire archive

**Output**: Email embeddings, topic clusters, similarity index, gem candidates.

### Tier 2 — Classification & Triage ($0.50 via batch API)
- Model: GPT-4o-mini or Gemini 2.0 Flash via Batch API
- Single LLM call per email, structured JSON output
- Category: personal / professional / transactional / newsletter / automated
- Keep/delete recommendation with reason
- Sentiment polarity + confidence
- Entity extraction (people, orgs, locations, dates, events)
- Importance score (1-10)
- Emotional weight flag
- Topic tags (1-3 from predefined taxonomy)

**Filtering rule**: Importance ≤3 AND transactional → delete pile. Importance ≥7 OR emotional flag → Tier 3. Everything between → store classification, skip deep analysis.

**Output**: Per-email classifications. ~2,000 emails flagged for deeper analysis. ~500 flagged as high-value.

### Tier 3 — Relationship & Temporal Analysis ($3.00)
- Model: Claude Haiku (bulk) or Sonnet (top 500)

**Entity Resolution** (mostly code, LLM for edge cases):
- Splink with Jaro-Winkler on names + emails
- Graph co-occurrence analysis on CC/thread participation
- Hebrew-English name matching via lookup table + transliteration + LLM fallback
- Output: Unified people directory with merged identities

**Relationship Profiling** (LLM-assisted):
- For each contact with >5 emails: classify type, depth, evolution
- Track greeting/closing style as primary formality indicator
- Per-relationship sentiment trajectory
- Score each contact: warmth, formality, reciprocity, longevity

**Era Detection** (statistics + LLM verification):
- Aggregate metadata into monthly multivariate time series
- Run ruptures PELT to detect 5-15 change points
- For each boundary: sample emails from both sides, LLM verifies and describes what changed
- Label each era with narrative description

**Gem Identification** (LLM-scored):
- Score candidates on: humor, emotional depth, historical significance, quotability
- Preserve original language (Hebrew subjects, jokes, meaningful fragments)

**Output**: People directory, life timeline with labeled eras, gem collection, per-relationship evolution.

### Tier 4 — Synthesis & Insight Generation ($2.00)
- Model: Claude Sonnet or Opus, ~50 mega-prompts via batch
- Relationship narratives ("Tell the story of your friendship with Maya over 12 years")
- Behavioral patterns (communication style, how writing evolved)
- Psychological insights (attachment patterns, conflict style, emotional expression)
- Era summaries (key people, events, themes, emotional tone per period)
- Cross-relationship patterns (who bridges eras, which relationships survived transitions)
- Cross-domain recurring patterns (Levels 7-9: archetypes, hidden connections, life motifs)

**Output**: The complete biographical intelligence — structured data + narratives.

---

## The 9 Depth Levels — Mailo's Unique Framework

### Surface Tier (Levels 1-3): What happened?
| Level | Name | Example | Technique | Cost |
|-------|------|---------|-----------|------|
| 1 | Basic | "Email from Maya, June 2010, camp supplies" | Header parsing | $0 |
| 2 | Surface | "Maya is a coordinator who sends logistics" | Aggregation across 10-20 emails | $0 |
| 3 | Understandable | "She's emailed you since 2007 across 2 programs" | Entity resolution + timeline query | $0 |

### Analytical Tier (Levels 4-6): What does it mean?
| Level | Name | Example | Technique | Cost |
|-------|------|---------|-----------|------|
| 4 | Continuous | "Appears in 9 periods. Tone shifts formal→warm" | PELT + sentiment scoring | Low |
| 5 | Deep | "Gave you increasing autonomy year over year" | Behavioral coding + sequential pattern mining | Medium |
| 6 | Indirect | "IDF letter = institutional advocacy at personal cost" | Tree-of-Thought hypothesis generation | Medium |

### Profound Tier (Levels 7-9): What deep truth does this reveal?
| Level | Name | Example | Technique | Cost |
|-------|------|---------|-----------|------|
| 7 | Very Deep | "Functions as professional identity mother-figure" | Cross-entity archetypal comparison | High |
| 8 | Surprising | "Her family connects to another key person" | Graph shortest-path + bridge detection | Medium |
| 9 | Recurring | "You seek mentor figures who give autonomy — same pattern across 3 domains" | Cross-domain pattern synthesis | High |

**Key principle**: Levels 1-3 are FREE (pure code). Levels 4-6 cost pennies per insight. Levels 7-9 are where the expensive LLM calls go — but they only run on the ~50 most significant relationships, not all 5,000 emails.

---

## Core Tools (All pip-installable)

| Function | Tool | Why this one |
|----------|------|-------------|
| Email parsing | Python `email` + `mailbox` stdlib | Handles MBOX, EML natively |
| Quoted text removal | Talon (Mailgun) | SVM-based signature/quote stripping |
| Entity resolution | Splink | Fellegi-Sunter on DuckDB, 1M records/minute, no training data |
| Contact graph | NetworkX | Community detection, centrality, shortest paths |
| Change-point detection | ruptures (PELT) | 5 lines of code, O(n), proven best performer |
| Topic modeling | BERTopic | Dynamic topics over time, works with short documents |
| Sentiment (English) | EmoRoBERTa / GoEmotions | 27 emotion categories, F1=0.92 for gratitude |
| Sentiment (Hebrew) | HeBERT (HebEMO) | 8 Plutchik emotions, F1=0.78-0.97 |
| Hebrew morphology | DictaBERT-seg | HuggingFace, single call |
| Embeddings | text-embedding-3-small OR paraphrase-multilingual-MiniLM-L12-v2 | $0.02/MTok or free local |
| Anomaly detection | Modified Z-score + Isolation Forest (scikit-learn) | 10-15 lines each |
| Relationship survival | lifelines (Kaplan-Meier, Cox) | Which relationships survive transitions |
| Seasonal patterns | STL decomposition (statsmodels) | Birthday patterns, holiday rhythms |
| Psychological profiling | Empath | 200+ categories, free, r=0.90 correlation with LIWC |
| LLM processing | Claude Batch API (Haiku→Sonnet→Opus tiered) | 50% batch discount + prompt caching |
| Storage | SQLite + ChromaDB (file-based, zero config) | Graduate to PostgreSQL + pgvector for production |

---

## What This Means For Ben's Archive (Right Now)

### Option A: Restart with the new pipeline
- Export dartaryan@gmail.com via Google Takeout (MBOX)
- Run Tiers 0-2 programmatically (~2 hours of processing, ~$0.50)
- This produces: classified emails, entity-resolved people directory, auto-detected eras, flagged gems — ALL in structured JSON
- Then walk through Tiers 3-4 era by era (much faster than current process because the foundation is already built)

### Option B: Migrate existing work + fill gaps
- Convert the 10+ completed eras from markdown → JSON schema
- Run the pipeline only on unprocessed eras
- Use existing validated data as training signal for the pipeline

### Option C: Build the product first, be your own first user
- Build the Tier 0-2 pipeline as a standalone tool
- Test it on your own archive
- Iterate until it produces good enough output that Tier 3-4 (the human-in-the-loop part) is fast and enjoyable rather than tedious
- Then package for others

---

## Build Timeline (Solo Developer)

| Week | What | Output |
|------|------|--------|
| 1-2 | Email ingestion, parsing, metadata extraction, SQLite store. Run PELT for auto-era-detection. | "Chapters of your life" timeline + contact frequency dashboard |
| 3-4 | Claude Batch API integration for structured extraction. Entity resolution with Splink. Contact graph with NetworkX + community detection. | Classified emails, unified people directory, social network map |
| 5-6 | BERTopic for topic evolution. Per-person longitudinal analysis. Sentiment scoring. Gem detection. | Topic timeline, relationship evolution profiles, gem collection |
| 7-8 | Deep analysis pipeline (Levels 5-6). Behavioral coding. Inference prompting. Agentic RAG for evidence gathering. | Behavioral insights, implicit meaning extraction |
| 9-10 | Network discovery (Level 8). Cross-domain synthesis (Level 9). Visualization layer. | Hidden connections, recurring life patterns, full biographical intelligence |

---

## The Product Vision (Scalable Mailo)

### For the user:
1. Connect your email (Gmail API or upload Takeout MBOX)
2. Mailo runs Tiers 0-2 automatically (minutes to hours depending on archive size)
3. You see: your life timeline auto-segmented into eras, your people ranked by significance, your emails classified
4. Mailo asks you smart questions: "Maya Wolk appears in 9 periods. She seems to be a mentor. Is that right?"
5. Every answer you give makes the model richer
6. As you engage, deeper levels unlock: behavioral patterns, relationship dynamics, hidden connections
7. You can write letters to people, explore your timeline, discover things about yourself

### For the business:
- Cowork plugin OR standalone web app
- Free tier: Tiers 0-1 (metadata analysis, basic timeline) — $0 cost
- Paid tier: Tiers 2-4 (LLM analysis, deep insights) — $5-10 cost per archive, charge $29-49
- Premium: Letter writing, narrative generation, printed timeline book — charge $99+

### The moat:
- The 9-level depth framework is novel IP
- The "statistics find, LLMs narrate" architecture is 10-100x cheaper than naive LLM-everything approaches
- The biographical intelligence layer (Levels 7-9) has no competitor
- Network effects: as more people use it, common entity resolution improves (opt-in)
