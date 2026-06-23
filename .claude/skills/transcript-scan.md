# /transcript-scan

Extract facts, hypotheses, action items, decisions, meetings, principal signals, and open questions from a transcript. Route confirmed items to the vault.

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

Read through the cleaned transcript and extract items across seven types.

### Cross-cutting: Salience tagging

Before classifying by type, watch for salience signals throughout the transcript. Any item that appears alongside the following words gets `salience: true` and a brief `salience_reason`:

**Trigger words:** "flag", "risk", "blocker", "concern", "milestone", "critical path", "strategic", "the bet", "make-or-break", "executive wants", "ELT wants", "board wants", "leadership wants"

Set `salience_reason` to a short phrase capturing why it was flagged (e.g., `"flagged as blocker"`, `"leadership priority"`).

---

### What to extract

**Facts** — definitive claims stated as true.
- Include: named numbers, percentages, dates, confirmed outcomes, unambiguous status statements.
- Certainty boosters — these words raise confidence to `high` even for unresolved speakers: "confirmed", "the data shows", "definitely", "we know", "no question", "my number", "I own that".
- Exclude: questions, speculation, "we should", wishes, softened language.

**Hypotheses** — working theories or beliefs stated with uncertainty.
- Trigger words: "I think", "I bet", "maybe", "probably", "my read is", "we could", "might", "could be", "perhaps", "I wonder", "I believe", "it seems", "I'd expect", "roughly", "my gut".

**Action Items** — commitments by a named person to do something.
- Trigger words: "I'll", "can you", "[name] will", "take the lead", "owns that", "action item", "next steps", "follow up", "make sure".
- Capture: who owns it, what the action is, and any deadline phrase: "by Friday", "before the review", "EOD", "this quarter", "due", "ahead of".
- Only extract when ownership is clear (a named person or "I").
- If an open question is assigned to someone with a deadline, extract as both an Open Question and an Action Item.

**Decisions** — closure on a question or direction.
- Explicit: "let's go with", "final call", "approved", "signed off", "locked", "agreed".
- Implied (assent-after-proposal): "sounds good", "works for me", "great, moving on", "unless anyone objects" followed by no objection, silence treated as agreement after a specific proposal.

**Meetings** — commitments to convene people.
- Explicit: "let's schedule X", "I'll send a calendar invite", "can we set up a call about X", "let's sync", "book time", "get [X] in a room", "regroup before".
- Implicit: "we need to align on X" when the other party agrees ("yes, let's do that", "agreed").

**Principal Signals** — statements of personal priority, constraint, or non-negotiable position.
- Trigger words: "I care about", "my priority", "non-negotiable", "the bar is", "I'm not willing to", "the constraint is".
- Note which lever the signal applies to: budget, headcount, roadmap, pricing, margin, deal, or other.

**Open Questions** — things explicitly unresolved or unknown.
- Trigger words: "we don't know", "TBD", "need to find out", "unclear", "let's figure out", "outstanding".
- If someone is assigned to answer it AND a deadline is mentioned, also extract as an Action Item.

---

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

A certainty booster (see Facts above) overrides `low` confidence to `high` even when authority is `unknown`.

**Claim domain** is the topic area of the claim (e.g., a claim about "Q2 cost-to-serve" belongs to finance/cost domain).

**Step C — Default for unknown authority**

Treat as hypothesis unless the claim contains named numbers, specific dates, project names, or named deliverables — then treat as fact (still with `authority: unknown`).

**Step D — Recall bias**

When unsure whether to extract, extract and flag for review rather than discard.

---

## Step 4 — Auto-routing

For every extracted item where `confidence: high` AND `authority: owner`:
- Facts → write directly to `vault/facts.json` using the schema defined in Step 6.
- Decisions → write directly to `vault/decisions.json` using the schema defined in Step 6.
- Do NOT present these to the user in Q&A — they are auto-confirmed.

---

## Step 5 — Interactive Q&A

Present remaining items one at a time, in order of appearance in the transcript.

Wait for user input before moving to the next item.

**Facts:**
```
[FACT?] "<claim>" (<speaker> — <authority>, <confidence>)
Source: "<source_span>"
→ confirm as [f]act, [h]yp, or [s]kip to review:
```

**Hypotheses:**
```
[HYP?] "<theory>" (<speaker> — <authority>, <confidence>)
Source: "<source_span>"
→ confirm as [f]act, [h]yp, or [s]kip to review:
```

**Action Items:**
```
[ACTION?] "<action>" (owner: <name>, deadline: <deadline or none>)
Source: "<source_span>"
→ confirm as [a]ction or [s]kip to review:
```

**Decisions:**
```
[DECISION?] "<what was decided>" (decided by: <name or group>)
Source: "<source_span>"
→ confirm as [d]ecision or [s]kip to review:
```

**Meetings:**
```
[MEETING?] "<purpose>" (convener: <name>)
Source: "<source_span>"
→ confirm as [m]eeting or [s]kip to review:
```

**Principal Signals:**
```
[SIGNAL?] "<what they said>" (<speaker> — lever: <lever>)
Source: "<source_span>"
→ confirm as [si]gnal or [s]kip to review:
```

**Open Questions:**
```
[QUESTION?] "<what's unclear>"
Source: "<source_span>"
→ confirm as [q]uestion or [s]kip to review:
```

Routing:
- `f` → `vault/facts.json`
- `h` → `vault/hypotheses.json`
- `a` → `vault/actions.json`
- `d` → `vault/decisions.json`
- `m` → `vault/meetings.json`
- `si` → `vault/signals.json`
- `q` → `vault/questions.json`
- `s` → add to skipped list (for review file)

---

## Step 6 — Write confirmed items

### Determine next sequence number

For each target file, read the existing JSON array and count IDs that start with today's date prefix (e.g., `fact-2026-06-23-`). The next NNNN = that count (zero-padded to 4 digits; first ID of the day is `0000`).

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
  "salience": false,
  "salience_reason": null,
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
  "salience": false,
  "salience_reason": null,
  "would_confirm": "<what evidence would confirm this — infer from context, or null>",
  "source_span": "<verbatim quote from transcript>",
  "source": "<filename>",
  "created": "YYYY-MM-DD"
}
```

### Action Items JSON item

```json
{
  "id": "action-YYYY-MM-DD-NNNN",
  "action": "<what needs to be done>",
  "owner": "<resolved name, or raw transcript name if unresolved>",
  "deadline": "<deadline phrase, e.g. 'by Friday', '2026-07-01', or null>",
  "status": "open",
  "salience": false,
  "salience_reason": null,
  "source_span": "<verbatim quote from transcript>",
  "source": "<filename>",
  "created": "YYYY-MM-DD"
}
```

### Decisions JSON item

```json
{
  "id": "decision-YYYY-MM-DD-NNNN",
  "decision": "<what was decided>",
  "decided_by": "<name or group, e.g. 'Sarah Klein', 'team'>",
  "context": "<brief context for the decision — infer from surrounding transcript>",
  "salience": false,
  "salience_reason": null,
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
  "salience": false,
  "salience_reason": null,
  "source_span": "<verbatim quote from transcript>",
  "source": "<filename>",
  "created": "YYYY-MM-DD"
}
```

### Principal Signals JSON item

```json
{
  "id": "signal-YYYY-MM-DD-NNNN",
  "signal": "<what they said>",
  "speaker": "<resolved name, or raw transcript name if unresolved>",
  "lever": "budget|headcount|roadmap|pricing|margin|deal|other",
  "salience": false,
  "salience_reason": null,
  "source_span": "<verbatim quote from transcript>",
  "source": "<filename>",
  "created": "YYYY-MM-DD"
}
```

### Open Questions JSON item

```json
{
  "id": "question-YYYY-MM-DD-NNNN",
  "question": "<what's unclear or unknown>",
  "raised_by": "<resolved name, or raw transcript name if unresolved>",
  "owner": "<person assigned to answer it, or null>",
  "salience": false,
  "salience_reason": null,
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

### [FACT? / HYP? / ACTION? / DECISION? / MEETING? / SIGNAL? / QUESTION?] <claim or purpose>
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
X facts, Y hypotheses, Z action items, W decisions, V meetings, U signals, T questions confirmed, N items → review
```

If a review file was written, add:

```
Review file: vault/review/YYYY-MM-DD-review.md
```
