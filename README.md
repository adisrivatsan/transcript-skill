# transcript-skill

Two Claude Code skills for extracting structured knowledge from meeting transcripts.

No Python. No pip install. Claude reads, extracts, and writes directly.

---

## Setup

```bash
git clone <repo-url> transcript-skill
cd transcript-skill
# Open Claude Code in this directory
```

---

## Quick Start

**1. Initialize the vault**

```
/transcript-init
```

Claude will ask for your project name and stakeholders (name, role, domains, aliases).  
This writes `vault/roster.yaml`.

**2. Drop a transcript into `transcripts/`**

Accepts `.vtt` (WebVTT) or `.txt` (plain `Speaker: text` format).

**3. Scan it**

```
/transcript-scan meeting.vtt
```

Claude extracts facts, hypotheses, and meeting requests, then walks you through Q&A for anything that needs confirmation.

---

## What Gets Extracted

| Type | What | Where |
|------|------|-------|
| Facts | Definitive claims stated as true | `vault/facts.json` |
| Hypotheses | Beliefs with uncertainty ("I think…", "maybe…") | `vault/hypotheses.json` |
| Meetings | Commitments to convene people | `vault/meetings.json` |

Items from roster owners speaking in their domain are auto-confirmed (no Q&A).  
Everything else gets routed interactively.

Skipped or ambiguous items go to `vault/review/YYYY-MM-DD-review.md`.

---

## Vault Structure

```
vault/
├── roster.yaml          # stakeholder config (edit directly to add people)
├── facts.json           # confirmed facts
├── hypotheses.json      # working theories
├── meetings.json        # meetings to schedule
└── review/              # YYYY-MM-DD-review.md files
```

---

## ID Format

`fact-2026-06-23-0000`, `hyp-2026-06-23-0001`, `mtg-2026-06-23-0000`

IDs are zero-padded and scoped to the day.

---

## Out of Scope

- Multiple vaults per repo
- Batch mode (all items → review without Q&A)
- Web UI
