# Mailo Deep Research Results — Summary of Key Findings
> These are condensed findings from 3 deep research tracks conducted March 13, 2026
> Full reports were provided by Ben from external deep research sessions

---

## Research 1: Biographical Intelligence Engine (9 Depth Levels)

### Architecture: 5-layer processing pipeline
- Layer 0: Ingestion (Apache Tika, Python mailbox — pure code)
- Layer 1: Atomic extraction (cheap LLM, ~$25-50 for 50K emails)
- Layer 2: Entity resolution + graph construction (Neo4j, Leiden community detection)
- Layer 3: Hierarchical summarization (RAPTOR trees — cluster→summarize→cluster)
- Layer 4: Cross-document synthesis (Agentic RAG with ReAct pattern)

### Key techniques per depth level:
- Level 1-2: Standard NER + email header parsing
- Level 3: Entity resolution across name variants (Palantir "Dynamic Ontology" approach, OCCRP's FollowTheMoney model)
- Level 4: Pattern-of-Life analysis + BERTopic dynamic topic modeling + sentiment scoring
- Level 5: Activity-Based Intelligence behavioral profiling + sequential pattern mining
- Level 6: CIA "Outside-In Thinking" + Tree-of-Thought hypothesis generation
- Level 7: Cross-entity comparative analysis + hierarchical clustering of behavioral profiles
- Level 8: Link analysis (i2 Analyst's Notebook approach) + bridge detection + shortest path
- Level 9: CIA "Alternative Futures Analysis" + cross-domain sequential pattern mining

### UX: 3 macro-tiers
- Surface tier (1-3): Cards, timelines, profiles
- Analytical tier (4-6): Scrollytelling dashboards, evidence trails
- Profound tier (7-9): Contemplative journal mode, requires opt-in

### Knowledge graph schema (Neo4j):
- Nodes: Person, Email, Organization, Topic, Event, LifePeriod
- Relationships carry temporal properties and behavioral metadata
- Email modeled as intermediary node (Neo4j best practice)

---

## Research 2: Personal Communication Data Pipeline

### Entity resolution stack:
1. Deterministic: exact email match + Gmail normalizations (dot, plus) → resolves 40-60%
2. Fuzzy: Soft TF-IDF with Jaro-Winkler (F1 ~0.89) + Double Metaphone + nickname dictionary
3. Graph co-occurrence: CC/thread co-appearance clustering (NetworkX)
4. Cross-lingual: Hebrew-English lookup table + phonetic comparison + LLM fallback
5. LLM disambiguation: only for 5-10% remaining uncertain cases (94.3% accuracy)
- **Tool**: Splink (pip install, Fellegi-Sunter on DuckDB, 1M records/minute)

### Relationship signals:
- Intimacy signals = 32.8% of tie-strength (Gilbert & Karahalios)
- Frequency = 19.7% — less important than people think
- Greeting/closing style = single most reliable content indicator
- Metadata alone handles 70-80% of relationship classification
- Microsoft's SNARF: FromTo count, ToFrom count, imbalance metrics

### Hebrew NLP (state of art 2025-2026):
- DictaBERT: prefix segmentation, morphological tagging, sentiment, NER
- NeoDictaBERT (2025): 4,096-token context, bilingual Hebrew+English variant
- DictaLM 3.0 (2025): 24B/12B/1.7B params, 65K context, retains >98% English capability
- Frontier LLMs: Claude achieves 93.34% on Hebrew classification
- Hebrew costs ~4x more tokens (BPE tokenizer splits each character)
- **Practical recommendation**: LLM-centric for mixed content, dedicated tools only where cost/precision demands

### Era detection:
- PELT algorithm (ruptures library): F1 0.652-0.864, O(n) complexity
- Feed: monthly email volume, contact Jaccard similarity, domain diversity, topic proportions
- 5-15 change points typically align with major life transitions
- LLM verification: sample emails from each side of detected boundary
- **No commercial product does this** — genuinely novel

### Cost for 5,000 emails:
| Tier | Cost |
|------|------|
| Structural extraction | $0.00 |
| Embedding + clustering | $0.04 |
| Classification (GPT-4o-mini batch) | $0.41-$0.75 |
| Relationship + temporal analysis | $2.30-$4.65 |
| Synthesis + narratives | $1.50-$3.00 |
| **Total** | **$4.25-$8.44** |

---

## Research 3: Behavioral Signal Extraction

### Platform signal → email equivalent:
- Netflix watch completion → thread follow-through depth
- Spotify skip rate → delete-without-reading rate per sender
- Amazon co-purchase → contact co-occurrence in CC/threads
- TikTok rewatch → re-reading old threads (emotionally significant)
- Google dwell time → time between open and reply

### Change-point detection algorithms:
- **PELT**: Best overall, O(n), `ruptures` library, 5 lines of code
- **CUSUM**: Detects gradual drifts (response times creeping up over months)
- **BOCPD**: Bayesian confidence ("95% probability shift happened in March 2018")
- **Prophet (Facebook)**: Handles seasonality + changepoints + irregular gaps
- **HMM**: Define 5-7 hidden states, decode most likely state sequence via Viterbi

### Personality inference:
- Kosinski et al.: 300 Facebook Likes match spouse's personality judgment accuracy
- Peters & Matz (2024, PNAS Nexus): GPT-4 achieves r=0.29-0.33 on Big Five from text (zero-shot)
- Schwartz et al.: specific linguistic markers per trait from 700M words
- **Empath** library: 200+ psych categories, r=0.90 correlation with LIWC, free
- GoEmotions: 27 emotion categories, EmoRoBERTa F1=0.92 for gratitude

### Anomaly detection:
- Modified Z-score: 10 lines, robust to outliers, flag |M|>3.5
- Isolation Forest: 15 lines scikit-learn, multi-dimensional anomalies
- Survival analysis (lifelines library): Kaplan-Meier for relationship death probability
- Hawkes processes: self-exciting communication patterns, who influences whom
- Key finding: transient relationships don't fade gradually — they cease abruptly

### Core principle:
**You don't need the LLM to find the anomalies — you need it to narrate them.**
Statistics detect the 70% volume drop. LLM reads the emails and explains "this is when you left Company X and joined Company Y."
