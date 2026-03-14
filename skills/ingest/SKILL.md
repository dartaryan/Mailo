---
name: ingest
description: >
  This skill should be used when the user provides new data to Mailo — uploading a file,
  pasting text, sharing a screenshot, or pointing to a file in the inbox/ folder. Trigger
  when the user says "I have new data", "process this", "new file", "ingest this",
  "here's my MBOX", "I exported my emails", or places any file in inbox/. Also trigger
  for WhatsApp exports, Google Timeline JSON, PDFs, CSVs, screenshots, or pasted text.
  Routes each data type to the correct processing skill.
version: 0.1.0
---

# Ingest — Data Router

## Description

The entry point for all new data entering Mailo. When the user provides any data source — a file, pasted text, a screenshot, or a reference to something in `inbox/` — this skill identifies what it is and activates the correct processing chain. The user never selects a skill manually; Ingest decides where data goes.

## Trigger Conditions

- User uploads or points to a file in `inbox/`
- User pastes text or shares a screenshot
- User says "I have new data", "process this", "new file", or similar
- A new file appears in the `inbox/` directory
- User shares a link to a data export

## Supported Input Types

| Type | Detection Method | Processing Chain |
|------|-----------------|-----------------|
| MBOX email archive | Extension `.mbox`, or file starts with `From ` | → **Sieve** skill (full 5-pass pipeline) |
| WhatsApp export | `.txt` file with `[date, time] Name: message` format | → Extract contacts, timeline, gems, relationships |
| Screenshot/image | `.png`, `.jpg`, `.jpeg` | → OCR/describe, extract behavioral signals, generate gap questions |
| Google Timeline | `.json` from Google Takeout location history | → Extract locations, travel patterns, cross-reference with email dates |
| PDF document | `.pdf` | → Extract text, identify type (letter? certificate? form?), route accordingly |
| CSV/spreadsheet | `.csv`, `.xlsx` | → Check if pre-processed email classification from previous triage |
| Text input | User pastes text directly | → Identify type: letter, memory, correction, answer to gap question |
| Existing Mailo data | Markdown files from previous Mailo sessions | → Migration: convert to JSON schema, merge into state |

## Step-by-Step Instructions

### 1. Identify the Data Type
When the user provides data:
1. Check the file extension or content format against the table above
2. If ambiguous (e.g., a `.txt` that could be a WhatsApp export or a letter), ask the user: "This looks like it could be [A] or [B]. Which is it?"
3. Confirm with the user: "I identified [type]. Found [N] items. Proceeding with [skill chain]."

### 2. Multi-Angle Extraction
For EVERY piece of data, regardless of type, extract from these angles:

- **Metadata**: Dates, times, names, addresses, file properties
- **Content surface**: What is this about? One-line summary.
- **Behavioral signals**: Time of day, language choice, formality level
- **Emotional reading**: Tone, sentiment, warmth vs. distance
- **Relational signals**: Who is mentioned? What relationships are implied?
- **What's NOT there**: What's missing? What questions does this raise? → Write to `gaps.json`

### 3. Route to Processing Chain
Based on identified type, activate the appropriate skill:
- MBOX → Sieve (the primary pipeline)
- All other types → Apply multi-angle extraction inline, then update relevant state files (people.json, timeline.json, gems.json, gaps.json, signal-log.json)

### 4. Handle Multiple Files
If the `inbox/` folder contains multiple files:
1. List all files with their types and sizes
2. Ask the user which to process first
3. Process one at a time unless the user requests batch processing

## State Files

| File | Access |
|------|--------|
| `state/meta.json` | READ + WRITE (update sources list, processing_log, stats) |
| `state/people.json` | WRITE (new people discovered) |
| `state/timeline.json` | WRITE (new events discovered) |
| `state/gems.json` | WRITE (meaningful fragments found) |
| `state/gaps.json` | WRITE (questions raised by missing info) |
| `state/signal-log.json` | WRITE (metadata intelligence extracted) |

## Rules and Constraints

- **Never process data without confirming the type** with the user if there's any ambiguity
- **Always report what was found**: "I identified [type]. Found [N] items. Proceeding with [skill chain]."
- **Large files (>50MB)**: Warn the user about processing time before starting
- **Never modify original files** in `inbox/`. Work on copies.
- **Never skip the multi-angle extraction** — even "boring" data has metadata signals
- **Respond in English** — even when the user writes in Hebrew. Preserve original Hebrew content (names, subjects, quotes) as-is.

## Output Format

After processing any input, report:
```
📥 Ingest Report
─────────────────
Type: [detected type]
Source: [filename or "pasted text"]
Items found: [N]
Processing chain: [skill name]
New people discovered: [N]
New events placed: [N]
Gap questions generated: [N]
Signals extracted: [N]
```
