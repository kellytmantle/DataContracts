# CLAUDE.md — Data Contracts Project

This file gives Claude context about this project so it can work effectively across sessions without needing to be re-briefed.

---

## What this project is

A documentation project producing a guide on how large organisations should manage change within data platforms. The core subject is **data contracts** — the formal agreements between data producers and consumers that govern how data is structured, delivered, and allowed to evolve.

The document is targeted at two audiences:
- **Data engineers** working within data platforms (technical depth expected)
- **Senior managers with semi-technical backgrounds** (concepts should be explained clearly, with examples)

Kelly's role is to provide the ideas and conspectus; Claude's role is to help document those ideas clearly, critique them, challenge where concepts would be difficult to implement in practice, and provide real-world examples where needed.

---

## Folder structure

```
/Data Contracts
├── CLAUDE.md                          ← this file
├── README.md                          ← minimal, not the main index
├── wikidocs/                          ← the main documentation
│   ├── index.md                       ← documentation index and reader guide
│   ├── 01-why-data-contracts.md
│   ├── 02-what-is-a-data-contract.md
│   ├── 03-versioning.md
│   ├── 04-change-patterns.md
│   ├── 05-producers-and-consumers.md
│   ├── 06-data-product-taxonomy.md
│   ├── 07-operating-principles.md     ← technical/data principles (immutability, SSoT, etc.)
│   ├── 08-governance-and-ownership.md
│   ├── 09-roles-and-responsibilities.md
│   ├── 10-platform-operating-model.md ← organisational operating model (governance, people, metrics)
│   └── Data Platform BCBS239 Considerations.md
├── critiques/                         ← critical reviews of the wikidocs
│   ├── 2026-05-08-wikidocs-critique.md
│   └── 2026-05-09-industry-critique.md
├── Dictation - Raw/                   ← raw voice dictation files (unprocessed)
└── Dictation - Corrected/             ← corrected versions of dictation files
```
├── about me/
│   └── writing-style.md               ← instructions for writing that doesn't sound like AI

---

## Conventions

- **All documents must be in Markdown format** — no Word, PDF, or other formats for wikidocs output.
- **Wikidocs are numbered sequentially** (01, 02, … 10, etc.) with kebab-case names.
- **Writing style:** clear, simple English. Avoid jargon where possible; explain it where unavoidable. No unnecessary padding.
- **Tone:** authoritative but accessible. The document should read as a credible, practical guide — not an academic paper or a vendor whitepaper.
- **Examples are important.** Abstract concepts should be grounded with concrete examples that make the idea real for both audiences.
- **Critique is part of the process.** Claude should actively challenge where concepts would be difficult to implement in practice, not just document them uncritically.

---

## Key distinctions in this project

- **07 — Operating Principles** covers *technical and data principles* (immutability, single source of truth, repeatability, data duplication rules).
- **10 — Platform Operating Model** covers *organisational principles* (governance, ownership, people, prioritisation, metrics). Created to keep technical and organisational concerns distinct.

---

## Writing style rules

The full writing style guide is in `about me/writing-style.md`. Claude must follow these rules when writing or editing any content in this project.

**Words and phrases to avoid:** delve, realm, tapestry, leverage, harness, vibrant, crucial, compelling, nuanced, transformative, actionable, unlock, empower, holistic, pivotal, seamless, cutting-edge, game-changer. Also avoid: "In conclusion…", "It is important to note…", "Furthermore", "Moreover", "Additionally", "Ever-evolving landscape", "Embark on a journey", "It goes without saying", "A testament to…", "Not just X, it's Y."

**Structural patterns to avoid:** uniform sentence length; the three-part list paragraph (topic sentence → three points → summary); mechanical introductions ("This document will discuss…") and conclusions ("In conclusion, this paper has shown…"); heavy transition words; short isolated paragraphs with no connective tissue; em dashes in any form.

**What good writing does:** uses specific, concrete details rather than vague adjectives; uses active voice; uses contractions; varies sentence rhythm; reflects a point of view and synthesises ideas rather than just summarising them; shows reasoning ("This suggests that…", "What this means in practice is…").

---

## Skills available

- **dictation-corrector** — processes raw voice dictation files from `Dictation - Raw/`, corrects transcription errors, and saves to `Dictation - Corrected/`.
- **wikidocs-critique** — reads all wikidocs files and produces a structured critique saved to `critiques/`.
