# Accuracy Framework
## Why AI-Powered Contract Extraction Is More Reliable Than Manual Entry

---

## The Central Argument

The instinctive reaction to automating billing data is: *"What if the AI gets it wrong?"*

The right question is: *"Is AI more accurate than a human reading a 10-page contract and typing the data into a billing system after processing their 8th contract of the day?"*

The answer is yes — when designed correctly.

Manual entry has three failure modes:
1. **Reading errors** — misreads $42,000 as $24,000
2. **Transcription errors** — reads correctly but types wrong value
3. **Fatigue errors** — misses a field entirely when processing high volumes

Our system eliminates failure modes 1 and 2 entirely. Failure mode 3 is addressed by making every required field mandatory — the form cannot be submitted with empty required fields.

The only remaining error vector is AI extracting a wrong value with high confidence. The three-layer model below is designed to catch this.

---

## Three-Layer Accuracy Model

### Layer 1 — AI Extraction with Confidence Scoring

The LLM reads the full contract and returns two outputs per field:
1. The extracted value
2. A confidence score (0–100%)

**How confidence is determined:**

| Contract Signal | Confidence |
|---|---|
| Field explicitly labelled: *"Annual Fee: $24,000"* | 90–100% |
| Field in a clearly formatted table or schedule | 85–95% |
| Field implied but not labelled directly | 70–84% |
| Field inferred from surrounding context | 50–69% |
| Field not found or ambiguous | 0–49% |
| Field not present in document | 0% (Not Found) |

**How confidence is actually produced (and its limits):**

LLMs do not natively output calibrated probability scores. In v1, confidence is self-reported by the model via structured prompting — the model is asked to rate its certainty per field based on how explicitly the value appears in the document. This is imperfectly calibrated by design.

This limitation is precisely why the architecture has three layers and not one: Layer 2 (human review) catches what miscalibrated confidence misses, and v2 replaces self-reported confidence with RAG validation against ground-truth plan data — a structurally more reliable signal.

**Why confidence scoring matters:**

Without confidence scoring, a pre-filled form looks equally correct whether the AI is 98% sure or 55% sure. The ops user has no signal about where to focus their review. With confidence scoring, uncertainty is surfaced explicitly — the user knows exactly which fields need attention and which can be trusted.

---

### Layer 2 — Human-in-the-Loop Review

**Design principle:** The ops user never reads the contract. They review a 10-field form.

This is not a compromise — it is a deliberate accuracy improvement.

| Dimension | Old Process | New Process |
|---|---|---|
| What ops reviews | 10-page contract | 10-field form |
| Cognitive load | High — reading + extracting + typing | Low — verify + correct |
| Time per contract | 30–60 minutes | 30–90 seconds |
| Error from fatigue | High (8th contract of the day) | None — reviewing, not extracting |
| Accountability | Ops owns the data they entered | Ops owns the data they approved |

**The blocking rule:**

Any field with confidence below 70% is flagged red and the Approve button is disabled until that field is manually confirmed. This means:
- No low-confidence value can silently flow into billing
- Ops explicitly owns every field they approve
- The audit trail shows exactly what was AI-extracted vs. manually confirmed

**What ops does with a red field:**
1. See the AI's extracted value (pre-filled, even if low confidence)
2. Read just that line of the contract if needed (not the whole document)
3. Correct if wrong, confirm if right
4. Time cost: 10–15 seconds per red field

---

### Layer 3 — Pre-Submission Validation Rules

After human review and before subscription creation, the system runs a final automated validation pass:

| Rule | What It Checks | Action on Fail |
|---|---|---|
| Customer exists | Customer ID found in billing system | Block — show error |
| Plan is active | Plan name in current plan catalogue | Flag red — block |
| Price in range | Billing amount within plan min/max | Flag red — block |
| Currency valid | ISO 4217 currency code | Flag red — block |
| Start date logic | Not more than 90 days in past | Flag red — require finance approval |
| End date logic | End date after start date | Flag red — block |
| No duplicate | No active subscription for same customer + plan | Warning — allow override |
| Billing frequency | Matches available frequencies for plan | Flag red — block |

These rules catch errors that even a careful human reviewer might miss — for example, a plan that was retired last month or a price that was manually corrected to a value outside the approved range.

---

## Document Type Accuracy Targets by Input

*These are design targets to be validated against a benchmark set of 50+ diverse contracts before launch — not measured results.*

| Document Type | Extraction Approach | Target Accuracy |
|---|---|---|
| Text-based PDF | Direct LLM extraction | 96–99% |
| DOCX | Direct LLM extraction | 96–99% |
| Scanned PDF (good quality) | OCR → LLM | 88–94% |
| Scanned PDF (poor quality) | OCR → LLM (with warning) | 75–85% |
| Image file (JPG/PNG, clear) | OCR → LLM | 85–92% |
| Image file (low resolution) | OCR → LLM (with warning) | 60–75% |
| Non-contract document | LLM extraction attempt | <50% — triggers fallback |

**Handling lower-accuracy inputs:**

For scanned and image documents, more fields will be flagged yellow or red — requiring more ops review. This is intentional. The system doesn't hide uncertainty; it surfaces it. Ops spends 2 minutes instead of 1, but the data quality is preserved.

---

## Accuracy Targets

| Metric | Target | Measurement Method |
|---|---|---|
| Required fields extracted correctly (no edit needed) | > 95% | Track field edit events in Mixpanel |
| Fields requiring ops correction | < 10% | Field edit event rate |
| False positives (flagged but actually correct) | < 15% | Track confirmed-without-edit on red/yellow fields |
| Billing errors post-activation | 0 increase vs. baseline | Monthly billing audit |
| Extraction failure rate (triggers manual fallback) | < 3% | Fallback event rate |

---

## Why This Is More Accurate Than Manual Entry at Scale

At low volume (1–2 contracts/day), a careful human can achieve high accuracy.

At scale (10–20 contracts/day), human accuracy degrades due to:
- Fatigue (concentration drops over repeated tasks)
- Time pressure (rushing through lower-priority contracts)
- Interruptions (multi-tasking between contract processing and other ops work)

Our system's accuracy does not degrade with volume. The 20th contract processed in a day is handled with identical accuracy to the first. This is the core reliability argument at scale.

---

## What We Do Not Automate (And Why)

| Decision | Why Human Keeps It |
|---|---|
| Final approval | Creates accountability and audit trail |
| Resolving ambiguous clauses | Requires legal or business judgement |
| Approving custom pricing exceptions | Requires sales/finance sign-off |
| Handling contract disputes | Relationship management — not a data problem |
| Interpreting non-standard payment terms | Legal risk if misclassified |

Automation handles data extraction and pre-population. Humans handle judgement, accountability, and exceptions.

---

*Accuracy Framework Author: Abinaya J | Version 1.0 | February 2026*
