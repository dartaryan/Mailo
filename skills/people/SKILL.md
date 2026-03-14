---
name: people
description: >
  This skill should be used when working with Mailo's people directory — the record of
  every person in the user's life. Trigger when the user says "tell me about [person]",
  "who is [name]?", "who are the most important people", "show me my people",
  "people directory", "merge [person A] and [person B]", or when the user provides
  corrections about a person. Also activates when new people are discovered during
  sieve or deep analysis.
version: 0.1.0
---

# People — People Directory Management

## Description

Manages the people directory — the evolving record of every person in the user's life as revealed through their data. Handles entity resolution (matching names and emails to the same person), relationship tracking across eras, and cross-referencing connections between people. This is where raw email addresses become rich biographical entries.

## Trigger Conditions

- New person encountered during sieve or deep analysis
- User asks "tell me about [person]" or "who is [name]?"
- User asks "who are the most important people in my [era/life/period]?"
- User provides corrections about a person
- User says "merge [person A] and [person B]"
- User says "show me my people" or "people directory"

## Step-by-Step Instructions

### 1. Entity Resolution (Matching People)

When a new name or email appears, check against existing entries in `state/people.json`:

1. **Exact email match** → same person (99% confidence)
2. **Gmail normalization**: Dots are insignificant (`john.doe@gmail` = `johndoe@gmail`), plus addressing (`user+tag@gmail` = `user@gmail`)
3. **Name fuzzy matching**: Jaro-Winkler similarity > 0.85 AND same context → likely same person
4. **Hebrew-English cross-matching**: Use transliteration patterns. Common pairs:
   - בן/Ben, יעל/Yael, גל/Gal, אור/Or, מיי/Mey
   - מאיה/Maya, נתן/Natan, אופירה/Ofira, דניאל/Daniel
5. **Context co-occurrence**: Two names appear in the same thread/CC list + time period → likely same social circle
6. **When uncertain**: Create entry with `status: "unresolved"` and add to `gaps.json`: "Is [name A] the same person as [name B]?"

### 2. Creating a Person Entry

Generate the next sequential ID (`person-NNN`) and create a full entry:

```json
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
      "active_until": "2009"
    }
  ],
  "relationship_to_owner": {
    "type": "mentor",
    "depth": "deep",
    "description": "Central mentor figure across all delegation programs.",
    "evolution": [
      { "period": "2007-2009", "nature": "authority/coordinator" },
      { "period": "2010-2014", "nature": "mentor giving autonomy" }
    ]
  },
  "era_appearances": ["era-01", "era-04", "era-06"],
  "connections": [
    { "person_id": "person-042", "type": "family_link", "note": "Husband is cousin of Yael" }
  ],
  "email_count": 47,
  "notes": "",
  "source_email_ids": []
}
```

### 3. Operations

- **Add person**: Generate next sequential ID, create entry, increment `next_id`, save
- **Merge persons**: Combine two entries — keep all emails, names, eras from both. Prefer `confirmed` data over `inferred`. Keep the lower person_id, redirect the other.
- **Update person**: Add new information (new email, new era, role change). Never overwrite `confirmed` data with `inferred` data.
- **Query person**: Search by name (fuzzy), email (exact), era, importance, role
- **Cross-era analysis**: "Show me people who appear in 3+ eras" → these are the most significant relationships
- **Relationship mapping**: Show how a person's role evolved over time

### 4. Importance Scoring

Scale of 1-5:
- **1 = Incidental contact**: Appeared in 1-2 emails, no meaningful interaction
- **2 = Acquaintance**: Appears occasionally, functional relationship
- **3 = Colleague/friend**: Regular contact, appears in multiple contexts
- **4 = Significant relationship**: Deep emotional or professional impact, spans multiple eras
- **5 = Life-defining person**: Shaped the user's identity, decisions, or trajectory

## State Files

| File | Access |
|------|--------|
| `state/people.json` | READ + WRITE (primary data store) |
| `state/timeline.json` | READ (to link people to eras) |
| `state/gaps.json` | WRITE (unresolved identity questions) |
| `state/meta.json` | WRITE (update total_people stat) |

## Rules and Constraints

- **IDs are permanent** — never reuse a `person_id`, even after merging
- **User corrections are final**: When the user corrects a person entry, set `status: "corrected"` and note what changed in `notes`
- **Never overwrite confirmed data with inferred data**
- **Generate gap questions** for ambiguous relationships: "Who is [name]? They appear in [N] emails from [period]."
- **Preserve Hebrew names**: `name_original` always stores the Hebrew version if available
- **Respond in English** — descriptions, roles, and relationship summaries are in English. Names preserved in original language.
- **Known family** (pre-populated context, not stored until confirmed in data):
  - Partner: Gal
  - Sister: Mey
  - Mother: Ofira
  - Stepfather: Natan (deceased)

## Output Format

When presenting a person to the user:
```
👤 [name_display] ([name_original])
   Importance: ★★★★☆ (4/5)
   First seen: [year] | Last seen: [year]
   Emails: [count]
   Eras: [list of era names]
   Role: [current/latest role and context]
   Relationship: [relationship_to_owner description]
   Evolution: [brief timeline of how the relationship changed]
   Connections: [links to other people]
```
