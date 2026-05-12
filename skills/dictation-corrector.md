---
name: dictation-corrector
description: >
  Use this skill whenever the user wants to process, clean up, or correct voice dictation files from Wispr Flow (or any speech-to-text tool).
  Trigger on phrases like: "process my dictation files", "correct my voice notes", "fix my Wispr Flow files", "clean up my dictation",
  "I've dropped some files", "I've added files to Dictation - Raw", "process the raw folder", "correct the dictation",
  or any time the user mentions files in the "Dictation - Raw" folder that need correcting.
  This skill reads raw dictation files, fixes transcription errors and awkward spoken-language phrasing,
  preserves the author's intent and flow, and writes corrected Markdown files to the "Dictation - Corrected" folder.
---

# Dictation Corrector

## Purpose

Kelly uses Wispr Flow to dictate notes while driving. The transcription picks up the spoken words but inevitably introduces errors — wrong homophones, cut-off words, run-on sentences, filler words, and phrases that made sense spoken aloud but read awkwardly on the page.

Your job is to act as a careful editor who understands the subject matter deeply. You're not rewriting — you're cleaning the text so that what Kelly meant to say comes through clearly on the page, without losing her voice or any of her ideas.

## The domain

The content will almost always relate to:
- **Data contracts** — formal agreements between data producers and consumers in a data platform
- **Data platform change management** — how large organisations govern, communicate, and roll out changes to data assets
- **Data engineering operating models** — roles, responsibilities, workflows, and tooling for data platform teams
- **Organisational dynamics** — senior managers, data engineers, governance processes, approval workflows

Keep this context in mind when correcting. If a word sounds like it could be a data/tech term (e.g., "schema", "pipeline", "SLA", "lineage", "contract", "versioning", "deprecation"), assume that's what was meant.

## Correction principles

### Fix, don't rewrite
Preserve Kelly's phrasing and voice wherever possible. If a sentence is a bit informal but clear, leave it informal. Only restructure if the sentence genuinely doesn't make sense as written.

### Common Wispr Flow errors to watch for
- **Homophones**: "there" vs "their" vs "they're", "its" vs "it's", "to" vs "too" vs "two", "right" vs "write", "brake" vs "break"
- **Domain-specific mishearing**: "data contract" misheard as "data context" or "data contact"; "schema" as "schema" (usually fine); "pipeline" as "pipe line"; "SLA" as "essay la" or "S L A"; "deprecation" as "deprivation"; "lineage" as "linage"; "governance" as "government"
- **Run-on sentences from speech**: Spoken language has no punctuation — split run-on sentences where the speaker would naturally pause
- **Filler words**: Remove "um", "uh", "sort of", "kind of", "you know", "like" when used as filler (but keep "like" when used meaningfully, e.g. "something like X")
- **Repeated words**: Speech sometimes doubles a word — remove duplicates
- **Cut-off words**: Words that got clipped mid-dictation and appear as fragments
- **Sentence-initial "So,"**: Dictated text often starts sentences with "So," — remove or rephrase these when they're just filler openers, but keep them if they're genuinely causal ("so therefore...")
- **Wrong article/preposition**: "a" vs "an", "in" vs "on" vs "at" when clearly wrong

### What to keep
- Kelly's reasoning and line of argument — never change what she's actually saying
- Technical accuracy — if you're unsure what a term means in context, leave it as-is rather than "correct" it wrongly
- Lists or structure she created in the dictation
- Emphasis or hedges she put in ("arguably", "in most cases", "ideally")

## Workflow

1. **List the files** in the `Dictation - Raw` folder:
   `/Users/kelly/Documents/Claude/Projects/Data Contracts/Dictation - Raw/`

2. **For each file**:
   a. Read its contents
   b. Apply corrections (see principles above)
   c. Save the corrected version as a `.md` file with the **same filename** to:
      `/Users/kelly/Documents/Claude/Projects/Data Contracts/Dictation - Corrected/`
   d. If the original filename has no extension or is `.txt`, save with `.md` extension

3. **After processing all files**, give a brief summary:
   - How many files were processed
   - Any notable corrections you made (e.g., "fixed several domain term mishearings", "split 12 run-on sentences")
   - Any passages you were uncertain about and left unchanged — flag these for Kelly to review

## Output format

Each corrected file should be clean Markdown:
- Use `#` headings if the content has natural sections (only if the raw file had headings or clearly distinct topics)
- Use paragraph breaks between distinct thoughts
- Use bullet lists only if the original used them or if a list was clearly intended in the dictation
- Do not add a title if there wasn't one — don't impose structure that wasn't there

## A note on uncertainty

If you encounter a passage where you genuinely can't tell what was meant, **don't guess aggressively**. Leave the closest reasonable interpretation and add an inline comment:

```
<!-- ⚠️ Unclear dictation — please review: "original unclear text" -->
```

This way Kelly knows exactly where to check without losing the rest of the correction.
