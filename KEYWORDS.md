# Extraction Keywords

These are the words and phrases `/transcript-scan` uses to classify items from a transcript.

---

## Hypothesis Trigger Words

A statement is extracted as a **hypothesis** when the speaker uses any of these:

| Phrase | Example |
|--------|---------|
| "I think" | "I think we can hit 15% growth" |
| "maybe" | "Maybe that's a pricing issue" |
| "probably" | "That's probably a Q3 problem" |
| "we could" | "We could expand into that segment" |
| "might" | "This might affect retention" |
| "perhaps" | "Perhaps the mix is shifting" |
| "I wonder" | "I wonder if the model accounts for churn" |
| "I believe" | "I believe the margin is closer to 40%" |
| "it seems" | "It seems like customers want bundling" |
| "I'd expect" | "I'd expect that to normalize" |

---

## Fact Indicators

A statement is extracted as a **fact** when it is definitive — no hedging, and includes one or more of:

- Named numbers or percentages ("revenue grew 12%")
- Specific dates ("we closed Q2 at…")
- Confirmed outcomes ("the board approved…")
- Unambiguous status statements ("we are live in 6 markets")

---

## Meeting Trigger Phrases

A statement is extracted as a **meeting commitment** when it includes:

**Explicit:**
- "let's schedule…"
- "I'll send a calendar invite"
- "can we set up a call about…"

**Implicit (only when the other party agrees):**
- "we need to align on X" + "yes, let's do that" / "agreed"

---

## Not Extracted

These are explicitly excluded:

- Questions
- Wishes or aspirations without commitment
- "We should…" statements (unless paired with an explicit agreement)
- Softened language that doesn't meet hypothesis or fact thresholds

---

## Customizing

To add project-specific trigger phrases (e.g., "our working assumption is", "the thesis is", "we're modeling that"), edit the **Hypotheses** section in `.claude/skills/transcript-scan.md` — find the line starting with `Trigger words:` and append your phrases.
