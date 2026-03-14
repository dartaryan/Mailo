# Mailo — Deep Research Prompts
> Created: March 13, 2026
> Usage: Copy-paste each prompt into Claude Deep Research (or any deep research tool). Bring results back to Mailo project for synthesis.

---

## Research 1: Behavioral Signal Extraction

```
I'm building a product called Mailo — a personal biographical intelligence engine that processes someone's email archive (thousands of emails spanning 10-20 years) and extracts behavioral patterns, personality insights, and relationship dynamics. Think of it as "what Netflix/Spotify/Amazon knows about you from your behavior — but applied to your personal email, and the insights go back to YOU instead of being used for ads."

I need to understand how major tech companies and recommendation systems extract personality and behavioral profiles from raw interaction data. Specifically:

1. IMPLICIT SIGNAL EXTRACTION: How do Netflix, Spotify, Amazon, Pinterest, TikTok, and Google infer personality traits, preferences, moods, and behavioral patterns from user actions (clicks, watches, purchases, scrolls, skips) rather than explicit self-reporting? What are the specific techniques — collaborative filtering, content-based filtering, implicit feedback modeling, session analysis?

2. TEMPORAL BEHAVIORAL PATTERNS: How do these systems detect behavioral changes over time? For example: "this user used to watch comedies but shifted to documentaries" or "this user shops more at night" or "this user's music taste changed after a life event." What algorithms detect drift, phase transitions, or seasonal patterns in user behavior?

3. PERSONALITY INFERENCE FROM DIGITAL TRACES: What does the research say about inferring Big Five personality traits, communication styles, emotional patterns, and psychological profiles from digital behavior? Include academic papers (like the Kosinski/Stillwell Facebook Likes study), industry approaches, and any open-source implementations.

4. FREQUENCY AND ABSENCE ANALYSIS: How do systems detect meaning in patterns of frequency (someone emails a lot at 2am, someone forwards everything to themselves, someone apologizes frequently) AND in absences (someone who used to email weekly suddenly stops)? What are the techniques for anomaly detection in personal behavioral data?

5. PRACTICAL ARCHITECTURE: What does the actual data pipeline look like? From raw event log (email sent, email received, attachment opened) to structured behavioral profile. What intermediate representations are used? Feature vectors? Embeddings? Time series?

I want concrete techniques, algorithms, papers, and architectural patterns — not high-level overviews. Focus on what's implementable by a solo developer using LLMs (Claude/GPT) as the analysis engine rather than building ML models from scratch. The key constraint is: the input is EMAIL data (sender, recipient, date, subject, body, attachments, thread structure) not click streams. So I need to understand how to ADAPT these techniques to email.

Deliverable: A structured report with specific techniques I can implement, ordered from simplest/highest-value to most complex. Include examples of what each technique would reveal when applied to a 15-year email archive.
```

---

## Research 2: Personal Communication Data Pipelines

```
I'm building a product that ingests a person's entire email archive (3,000-10,000 emails spanning 10-20 years, mixed Hebrew and English) and automatically produces:
- A people directory with evolving relationship profiles
- A life timeline organized into "eras" (life periods)
- Classified emails (keep/delete with reasons)
- Behavioral patterns and psychological insights
- Preserved "gems" (jokes, meaningful quotes, emotional moments)

I need to understand how modern NLP and data engineering systems process unstructured personal communication data into structured behavioral profiles. Specifically:

1. ENTITY RESOLUTION IN PERSONAL EMAIL: How do you identify that "wolk.maya@gmail.com", "mayawo@netvision.net.il", "Maya", and "מאיה" are all the same person? What are the best approaches for entity resolution across email addresses, name variations, nicknames, and multilingual names? Include techniques used by personal CRMs (Clay, Dex), email clients, and academic research.

2. RELATIONSHIP EXTRACTION AND EVOLUTION: How do you automatically determine what role someone plays in the email owner's life — and how that role changes over time? Techniques for: classifying relationship type (family, friend, colleague, service provider), detecting relationship depth (casual vs. deep), tracking relationship evolution (formal → warm → distant). What signals in email data indicate these things? (Tone, frequency, greeting style, time of day, CC patterns, thread length)

3. MULTILINGUAL NLP FOR PERSONAL TEXT: Best practices for processing mixed Hebrew-English email content. Hebrew NLP challenges (right-to-left, no vowels, morphological complexity). How to handle code-switching within a single email. Available Hebrew NLP models and tools (2024-2026 state of the art). How LLMs (Claude, GPT) handle Hebrew compared to dedicated Hebrew NLP.

4. TEMPORAL SEGMENTATION — "ERA DETECTION": How do you automatically segment a person's life into meaningful periods from their communication data? Techniques for detecting phase transitions: change in frequent contacts, change in email patterns, change in topics, change in location references. This is similar to "change point detection" in time series — what's the state of the art when applied to communication data?

5. SENTIMENT AND TONE TRACKING OVER TIME: How do you track emotional tone across years of correspondence? Detecting shifts from formal to informal, from warm to distant, from frequent to silent. What does the research say about sentiment analysis applied to personal email (not customer reviews or social media)?

6. SCALE AND COST: Processing 5,000 emails through an LLM is expensive. What are the best strategies for tiered processing? (Rule-based pre-filtering → cheap model for triage → expensive model for deep analysis). What can be done with regex/heuristics vs. what requires LLM intelligence? How do companies like Superhuman, Shortwave, or Google process email at scale?

Deliverable: An architectural blueprint for a personal email processing pipeline. Show me the stages, what happens at each stage, what tools/techniques are used, and what the output looks like. Prioritize approaches that work with LLMs as the core engine (not custom ML training). Include cost estimates for processing 5,000 emails through different approaches.
```

---

## Research 3: Progressive Context Layering & Knowledge Discovery

```
I'm building a biographical intelligence system that processes someone's email archive and produces layered insights — from surface-level facts to deep psychological patterns. The core idea is that insight has DEPTH LEVELS:

1. Basic — "This email is from Maya, dated 2010, about camp supplies"
2. Surface — "Maya is a coordinator who sends camp logistics"  
3. Understandable — "She's been emailing you since 2007 across two programs"
4. Continuous — "She appears in 9 life periods. Tone shifts from formal to warm"
5. Deep — "She gave you increasing autonomy year over year"
6. Indirect — "Her writing an IDF letter for you suggests institutional advocacy"
7. Very deep — "She functions as a professional identity mother-figure"
8. Surprising — "Her family is connected to another key person — your network is more intertwined than you realized"
9. Recurring — "You seek mentor figures who give autonomy. Same pattern across 3 different life domains"

I need to understand how knowledge discovery systems, intelligence analysis frameworks, and investigative tools build this kind of layered, deepening understanding from large document collections. Specifically:

1. INTELLIGENCE ANALYSIS METHODS: How do CIA/Mossad/MI6 analysts go from thousands of documents to actionable insights? Structured Analytic Techniques (SATs), Analysis of Competing Hypotheses (ACH), link analysis, timeline reconstruction, pattern-of-life analysis. What's applicable to personal biographical analysis? What tools exist (Palantir, i2 Analyst's Notebook, Maltego)?

2. INVESTIGATIVE JOURNALISM TECHNIQUES: How do journalists like those at Bellingcat, ICIJ (Panama Papers), or ProPublica process massive document dumps to find stories? Their workflows for entity extraction, cross-referencing, timeline building, and pattern detection. Open-source intelligence (OSINT) techniques applicable to personal email.

3. KNOWLEDGE DISCOVERY IN DATABASES (KDD): The academic field of extracting non-trivial, previously unknown, and potentially useful patterns from data. What techniques are relevant to personal communication data? Association rule mining, sequential pattern mining, anomaly detection, clustering.

4. PROGRESSIVE DISCLOSURE IN UX: How do information systems present layered complexity to users? The concept of "details on demand" — starting with a summary, letting the user drill deeper. How does this apply to presenting biographical insights? Examples from data journalism, interactive documentaries, or exploratory data tools.

5. LLM-BASED KNOWLEDGE SYNTHESIS: How are LLMs being used (2024-2026) to synthesize insights across large document collections? RAG architectures for personal data, multi-document summarization, cross-document entity and relationship tracking, iterative deepening (first pass → second pass → synthesis). What are the actual prompting strategies for getting an LLM to produce layered analysis rather than flat summaries?

6. GRAPH-BASED INSIGHT DISCOVERY: How do knowledge graphs and network analysis reveal hidden connections in personal data? Community detection (finding clusters of people), centrality analysis (who's most important), bridge detection (who connects different social circles), temporal network analysis (how the network evolves).

Deliverable: A framework for how Mailo should build its "insight engine" — the component that goes beyond classification to produce genuine biographical intelligence. Include concrete techniques at each depth level (1-9), with examples of what each technique would reveal when applied to a 15-year personal email archive. Prioritize approaches implementable with LLMs, not custom ML.
```

---

## How to Use These

1. Run each prompt as a separate Deep Research session
2. Save the full output from each
3. Bring all three back to our Mailo project conversation
4. We'll synthesize the findings into a concrete architecture and implementation plan

The three research tracks together answer:
- **Research 1**: What signals to look for (the WHAT)
- **Research 2**: How to extract them from email (the HOW)  
- **Research 3**: How to build layered insight from those signals (the SO WHAT)
