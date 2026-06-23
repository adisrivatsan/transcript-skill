# /transcript-scan

Extract facts, hypotheses, and meetings from a transcript. Route confirmed items to the vault.

**Invocation:** `/transcript-scan <filename>`

The file must be in `transcripts/`. Pass filename only (e.g., `meeting.vtt`) or a full path.

---

## Step 1 — Read inputs

Read:
- `vault/roster.yaml` — load `project_name` and `people` list (names, roles, domains, aliases)
- `transcripts/<filename>` — load the raw transcript text

If either file is missing, stop and report the specific missing file.

---

## Step 2 — VTT preprocessing (`.vtt` files only)

If the filename ends in `.vtt`, clean the raw text before extraction:

1. Remove the `WEBVTT` header line.
2. Remove numeric cue identifier lines (lines containing only a number, e.g., `1`, `47`).
3. Remove timestamp lines — any line matching `HH:MM:SS.mmm --> HH:MM:SS.mmm` (with optional trailing alignment metadata).
4. Remove blank lines between cues.
5. Merge consecutive lines that belong to the same speaker's cue into one `Speaker: text` line (join with a space).

Output: one `Name: text` line per speaker turn.

`.txt` files are already in `Name: text` format — skip this step entirely.

If a `.txt` file does not appear to have `Speaker: text` speaker labels (e.g., no colons, unstructured prose), warn the user: "This file doesn't appear to have speaker labels. Continue extraction anyway? (yes/no)" — stop if they say no.

---

## Step 3 — Extraction

Read through the cleaned transcript and extract items.

### What to extract

**Facts** — definitive claims stated as true.
- Include: named numbers, percentages, dates, confirmed outcomes, unambiguous status statements.
- Exclude: questions, speculation, "we should", wishes, softened language.

**Hypotheses** — working theories or beliefs stated with uncertainty.
Trigger words: "I think", "maybe", "probably", "we could", "might", "perhaps", "I wonder", "I believe", "it seems", "I'd expect".

**Meetings** — commitments to convene people.
- Explicit: "let's schedule X", "I'll send a calendar invite", "can we set up a call about X".
- Implicit: "we need to align on X" when the other party agrees ("yes, let's do that", "agreed").

### Authority assignment (apply per extracted item)

**Step A — Alias resolution**

Match the speaker name against all roster `aliases` fields (case-insensitive). A person named "Sarah Klein" with `aliases: [Sarah, SK]` matches "Sarah Klein", "Sarah", or "SK".

- If the name matches exactly one person: resolved.
- If ambiguous (matches multiple people): flag `needs_review: true` on the item.
- If no match: speaker is unresolved.

**Step B — Assign authority and confidence**

| Condition | authority | confidence |
|-----------|-----------|------------|
| Resolved AND claim domain ∈ speaker's `domains` | `owner` | `high` |
| Resolved AND claim domain ∉ speaker's `domains` | `non_owner` | `low` |
| Unresolved | `unknown` | `low` |

**Claim domain** is the topic area of the claim (e.g., a claim about "Q2 cost-to-serve" belongs to finance/cost domain).

**Step C — Default for unknown authority**

Treat as hypothesis unless the claim contains named numbers, specific dates, project names, or named deliverables — then treat as fact (still with `authority: unknown`).

**Step D — Recall bias**

When unsure whether to extract, extract and flag for review rather than discard.

---

## Step 4 — Auto-routing

For every extracted item where `confidence: high` AND `authority: owner`:
- Write directly to `vault/facts.json` using the schema defined in Step 6.
- Do NOT present these to the user in Q&A — they are auto-confirmed.

---

## Step 5 — Interactive Q&A

Present remaining items one at a time, in order of appearance in the transcript.

**For facts and hypotheses:**

For items extracted as facts:
```
[FACT?] "<claim>" (<speaker> — <authority>, <confidence>)
Source: "<source_span>"
→ confirm as [f]act, [h]yp, or [s]kip to review:
```

For items extracted as hypotheses:
```
[HYP?] "<theory>" (<speaker> — <authority>, <confidence>)
Source: "<source_span>"
→ confirm as [f]act, [h]yp, or [s]kip to review:
```

Wait for user input before moving to the next item.

Routing:
- `f` → write to `vault/facts.json`
- `h` → write to `vault/hypotheses.json` (if originally extracted as a fact, demote it)
- `s` → add to skipped list (for review file)

**For meetings:**

```
[MEETING?] "<purpose>" (convener: <name>)
→ confirm as [m]eeting or [s]kip to review:
```

Routing:
- `m` → write to `vault/meetings.json`
- `s` → add to skipped list

---

## Step 6 — Write confirmed items

### Determine next sequence number

For each target file (`facts.json`, `hypotheses.json`, `meetings.json`):
1. Read the existing JSON array (it may be empty `[]`).
2. Count IDs that start with today's date prefix (e.g., `fact-2026-06-23-`).
3. The next NNNN = that count (zero-padded to 4 digits; first ID of the day is `0000`).

Note: Auto-routed items written in Step 4 are already in the file. Re-read the file after Step 4 completes to get an accurate count before assigning new IDs.

### Facts JSON item

```json
{
  "id": "fact-YYYY-MM-DD-NNNN",
  "claim": "<claim text>",
  "stated_by": "<resolved name, or raw transcript name if unresolved>",
  "authority": "owner|non_owner|unknown",
  "confidence": "high|low",
  "needs_review": false,
  "source_span": "<verbatim quote from transcript>",
  "source": "<filename>",
  "created": "YYYY-MM-DD"
}
```

`needs_review` becomes `true` when the alias resolution step flags an ambiguous speaker.

### Hypotheses JSON item

```json
{
  "id": "hyp-YYYY-MM-DD-NNNN",
  "theory": "<hypothesis text>",
  "held_by": "<resolved name, or raw transcript name if unresolved>",
  "authority": "owner|non_owner|unknown",
  "confidence": "high|low",
  "needs_review": false,
  "would_confirm": "<what evidence would confirm this — infer from context, or null>",
  "source_span": "<verbatim quote from transcript>",
  "source": "<filename>",
  "created": "YYYY-MM-DD"
}
```

### Meetings JSON item

```json
{
  "id": "mtg-YYYY-MM-DD-NNNN",
  "purpose": "<what the meeting is for>",
  "convener": "<name of person who proposed it>",
  "attendees": ["<name>", "<name>"],
  "target_timing": "<any mentioned timing, e.g., 'next week', '2026-07-01', or null>",
  "status": "pending",
  "source_span": "<verbatim quote from transcript>",
  "source": "<filename>",
  "created": "YYYY-MM-DD"
}
```

Append new items to the existing array and write the file back.

---

## Step 7 — Write review file

If any items were skipped OR any alias flags (`needs_review: true`) were raised, write `vault/review/YYYY-MM-DD-review.md` (today's date).

For any item extracted under Step D's recall-bias rule (borderline items), add a brief entry under "Near-Discard Notes" explaining why it was included rather than discarded.

If the file already exists for today, append to it.

```markdown
# Review — YYYY-MM-DD

Source: <filename>

## Skipped Items

### [FACT? / HYP? / MEETING?] <claim or purpose>
- Speaker: <name>
- Authority: <authority>
- Source: "<source_span>"

## Alias Flags

### <raw name as it appeared in transcript>
- Possible matches: <name1>, <name2>
- Context: "<surrounding text>"

## Near-Discard Notes

<Items that were borderline — extracted but almost excluded — with a brief rationale for including them.>
```

---

## Step 8 — Summary

Print exactly:

```
X facts confirmed, Y hypotheses confirmed, Z meetings confirmed, N items → review
```

If a review file was written, add:

```
Review file: vault/review/YYYY-MM-DD-review.md
```
