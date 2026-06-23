# Extraction Keywords

These are the words and phrases `/transcript-scan` uses to classify items from a transcript. Seven extraction types plus a cross-cutting salience tag.

---

## Salience Tag (cross-cutting)

Any item extracted alongside these words gets flagged `salience: true`:

| Phrase | Notes |
|--------|-------|
| "flag" | General flag for attention |
| "risk" | Risk to plan or timeline |
| "blocker" | Blocking progress |
| "concern" | Raised concern |
| "milestone" | Key milestone reference |
| "critical path" | On the critical path |
| "strategic" | Strategic significance |
| "the bet" | Core strategic bet |
| "make-or-break" | High-stakes item |
| "executive wants" / "ELT wants" / "board wants" / "leadership wants" | Senior stakeholder priority |

---

## Facts

Extracted when a claim is **definitive** — no hedging — and includes at least one of:
- Named numbers or percentages
- Specific dates
- Confirmed outcomes
- Unambiguous status statements

**Certainty boosters** — these raise confidence to `high` even for unresolved speakers:

| Phrase |
|--------|
| "confirmed" |
| "the data shows" |
| "definitely" |
| "we know" |
| "no question" |
| "my number" |
| "I own that" |

---

## Hypotheses

Extracted when a speaker uses **hedging or uncertainty language**:

| Phrase | Example |
|--------|---------|
| "I think" | "I think we can hit 15% growth" |
| "I bet" | "I bet that's a retention issue" |
| "maybe" | "Maybe that's a pricing problem" |
| "probably" | "That's probably a Q3 problem" |
| "my read is" | "My read is the market is softening" |
| "we could" | "We could expand into that segment" |
| "might" | "This might affect retention" |
| "could be" | "Could be a sequencing issue" |
| "perhaps" | "Perhaps the mix is shifting" |
| "I wonder" | "I wonder if the model accounts for churn" |
| "I believe" | "I believe the margin is closer to 40%" |
| "it seems" | "It seems like customers want bundling" |
| "I'd expect" | "I'd expect that to normalize" |
| "roughly" | "Roughly 20% of the base" |
| "my gut" | "My gut says we're underpriced" |

---

## Action Items

Extracted when **a named person commits to do something**. Ownership must be clear.

| Phrase | Notes |
|--------|-------|
| "I'll" | Speaker self-commits |
| "can you" | Directed request to named person |
| "[name] will" | Third-party assignment |
| "take the lead" | Ownership assignment |
| "owns that" | Explicit ownership |
| "action item" | Explicit call-out |
| "next steps" | Summary of actions |
| "follow up" | Follow-up commitment |
| "make sure" | Directive with implied owner |

**Deadline signals** (captured on the action item when present):

| Phrase |
|--------|
| "by Friday" |
| "before the review" |
| "EOD" |
| "this quarter" |
| "due" |
| "ahead of" |

---

## Decisions

Extracted when a question or direction is **closed out**.

**Explicit:**

| Phrase |
|--------|
| "let's go with" |
| "final call" |
| "approved" |
| "signed off" |
| "locked" |
| "agreed" |

**Implied (assent-after-proposal):**

| Phrase | Condition |
|--------|-----------|
| "sounds good" | After a specific proposal |
| "works for me" | After a specific proposal |
| "great, moving on" | After a specific proposal |
| "unless anyone objects" | No objection follows |

---

## Meetings

Extracted when people **commit to convene**.

**Explicit:**

| Phrase |
|--------|
| "let's schedule" |
| "I'll send a calendar invite" |
| "can we set up a call" |
| "let's sync" |
| "book time" |
| "get [X] in a room" |
| "regroup before" |

**Implicit** (only when the other party agrees):

| Phrase | Condition |
|--------|-----------|
| "we need to align on X" | Other party says "yes, let's" or "agreed" |

---

## Principal Signals

Extracted when someone states a **personal priority, constraint, or non-negotiable**.

| Phrase | Notes |
|--------|-------|
| "I care about" | Priority statement |
| "my priority" | Priority statement |
| "non-negotiable" | Hard constraint |
| "the bar is" | Minimum threshold |
| "I'm not willing to" | Hard limit |
| "the constraint is" | Named constraint |

**Levers** (note which applies): budget, headcount, roadmap, pricing, margin, deal, other

---

## Open Questions

Extracted when something is **explicitly unresolved**.

| Phrase |
|--------|
| "we don't know" |
| "TBD" |
| "need to find out" |
| "unclear" |
| "let's figure out" |
| "outstanding" |

If the question is **assigned to someone** AND a **deadline is mentioned**, also extract as an Action Item.

---

## Not Extracted

Explicitly excluded:
- Questions without a resolution (unless they meet Open Question criteria)
- Wishes or aspirations without commitment ("we should…" without agreement)
- Speculation that doesn't meet hypothesis threshold
- Softened language below hypothesis trigger level

---

## Customizing

To add project-specific phrases, edit `.claude/skills/transcript-scan.md` — find the relevant "Trigger words:" line in Step 3 and append your phrases.
