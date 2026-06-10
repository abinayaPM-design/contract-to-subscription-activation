# Edge Cases & Tradeoffs
## Contract-to-Subscription Activation Engine

---

## Edge Cases

### Document Format Edge Cases

| Edge Case | Risk | Handling |
|---|---|---|
| Scanned PDF (image-based) | LLM cannot read images directly | OCR pre-processing before LLM extraction. Warn ops if OCR quality score is low. |
| Image file (JPG/PNG) | Same as scanned PDF | Same OCR pre-processing. Show confidence reduction warning. |
| Password-protected PDF | Cannot be processed at all | Reject on upload: "Please upload an unlocked version of this contract." |
| Corrupted file | Processing fails | Reject on upload with generic error. Offer re-upload. |
| Very large file (50+ pages) | Slow extraction, high token cost | Process in full. Show "Large document — may take up to 30 seconds." Set 20MB file size cap. |
| Multi-file upload | Ambiguity on which is authoritative | v1: single file only. Show error if multiple selected. v2: allow amendment attachment. |
| Non-English contract | LLM may hallucinate translations | Detect language. If non-English: warn ops, flag all fields red, offer manual entry. |
| Handwritten contract | OCR fails on cursive | Reject with guidance: "Please upload a typed version or enter details manually." |
| Heavily formatted PDF (tables, columns) | Field location ambiguous | LLM handles layout variation. Lower confidence expected — more fields flagged yellow. |
| Contract embedded in email thread | Extraneous text confuses extraction | LLM instructed to focus on contract body. Confidence reduced for embedded documents — warn ops. |

---

### Data Edge Cases

| Edge Case | Risk | Handling |
|---|---|---|
| Ramp pricing (Year 1: $10K, Year 2: $15K) | Single billing amount field insufficient | Extract all pricing tiers. Flag as "Ramp Pricing Detected." Add notes field for ops. |
| Multi-currency contract | Ambiguity on billing currency | Extract all currencies mentioned. Show dropdown: "Which currency should billing use?" |
| Backdated start date > 90 days | Revenue recognition complexity | Flag red. Require finance approval. Show clear warning to ops. |
| Future start date > 12 months | Must queue subscription creation | Extract date. Flag for ops attention. Show "Activation scheduled for [date]" on confirmation. |
| Free trial before billing starts | Contract start ≠ billing start | Extract both dates separately: contract_start_date and billing_start_date. |
| Volume-based / usage pricing | No fixed billing amount | Flag billing amount as "Usage-Based." Route to manual entry for that field only. |
| Auto-renewal with price increase | Renewal price differs from initial | Extract both. Flag "Renewal pricing differs from initial term." Add to notes. |
| Discount with expiry date | Discount applies only for N months | Extract discount + expiry. Surface in notes field for ops to verify in billing system. |
| Early termination clause | May affect subscription term | Extract if present. Flag as special term. No automated action. |
| Co-term contract (mid-cycle start) | Prorated first invoice | Flag start date and note "Co-term detected — verify proration in billing system." |
| Multi-product contract | Multiple plan lines in one contract | v1: extract primary plan only, flag if multiple plans detected. v2: multi-line extraction. |

---

### System Edge Cases

| Edge Case | Risk | Handling |
|---|---|---|
| Billing API timeout | Subscription not created, ops thinks it was | Never auto-retry silently. Show explicit failure state. Ops must manually re-trigger. Show "Activation Failed — your review data is saved." |
| Billing API down | Same as timeout | Same handling. Show estimated downtime if available. |
| Duplicate contract upload (same file) | Duplicate subscription | Hash-check on upload. Warn if file processed in last 30 days. Show previous subscription ID. |
| Similar contract (different file, same customer + plan) | Duplicate subscription | Post-extraction check: warn if active subscription found for same customer + plan. |
| Customer doesn't exist in billing system | Subscription creation fails | Validate customer ID before showing review form. Block with: "Customer not found — create in billing system first." |
| Plan no longer active in catalogue | Subscription creation fails | Validate plan name against catalogue at extraction time. Flag red immediately. |
| Two ops users opening same contract | Race condition — duplicate subscription | Lock contract for review once opened. Show "In review by [name]" to second user. Auto-release after 30 min inactivity. |
| Session timeout mid-review | Form data lost, ops must start over | Auto-save form state every 30 seconds. Restore on page reload. Show "Draft restored." |
| Internet drops mid-approval | Approval lost | Detect offline state. Queue approval for submission when connection restores. Show "You're offline — approval will submit when reconnected." |
| Extraction returns fields in wrong language | e.g. date formats, currency symbols | Normalise all extracted values to standard formats (ISO dates, ISO currency codes) before display. |

---

## Key Tradeoffs

### Tradeoff 1 — Always Require Human Review vs. Full Automation

**Chosen:** Always require human review (ops approves every contract)

**Rejected:** Auto-create subscription if all fields above 95% confidence

**Reasoning:**
At 95% confidence, a 5% error rate on billing data sounds small. At 20 contracts/month, that is 1 billing error per month. At 100 contracts/month, it is 5 errors per month — each requiring correction, customer communication, and potential credit. The 30-second review step eliminates this entirely. Full automation is a v2 consideration once extraction accuracy is validated at scale over 1,000+ contracts.

---

### Tradeoff 2 — Confidence Blocking Threshold at 70% vs. Other Values

**Chosen:** Block approval if any field below 70%

**Rejected Option A:** Block at 50% (more permissive)
**Rejected Option B:** Block at 85% (more conservative)

**Reasoning:**
70% is the inflection point where AI errors become meaningfully likely. At 50%, too many incorrect values pass through with only a yellow warning — ops may not review them carefully. At 85%, too many correct values are flagged red, creating unnecessary friction and slowing ops down. 70% balances accuracy and speed optimally. This threshold should be revisited after 500 contracts are processed and false positive/negative rates are measured.

---

### Tradeoff 3 — Who Can Upload Contracts

**Chosen:** Any user (sales rep or ops) can upload. Ops must review and approve.

**Rejected:** Only ops can upload

**Reasoning:**
Sales reps are the fastest path to upload — they have the signed contract immediately after the call and are highly motivated to get the customer live quickly. Restricting upload to ops adds a handoff delay (email contract → ops uploads → ops reviews). Accountability is fully preserved at the review and approval step regardless of who uploads.

---

### Tradeoff 4 — Finance Approval Trigger

**Chosen:** Configurable dollar threshold (default $50,000 annual contract value)

**Rejected Option A:** Finance approval for all contracts
**Rejected Option B:** No finance approval at all

**Reasoning:**
Requiring finance approval for a $500/month contract is friction with no proportionate risk reduction. Removing finance approval entirely exposes the company to pricing errors on high-value deals. A configurable threshold gives the business the right risk/speed tradeoff. The default of $50,000 ACV covers typical enterprise deal thresholds — adjustable by admin.

---

### Tradeoff 5 — Fallback on AI Failure

**Chosen:** Show blank manual entry form

**Rejected:** Reject upload, ask user to try again

**Reasoning:**
Rejection creates a dead end — the ops user has a contract to process and nowhere to go. A manual entry form means the workflow never breaks. The product degrades gracefully: fast when AI works, slower but functional when it doesn't. This is especially important for scanned documents from smaller customers who may not have digital contracts.

---

### Tradeoff 6 — Single File vs. Multi-File Upload (v1)

**Chosen:** Single file only in v1

**Rejected:** Allow contract + amendment upload simultaneously

**Reasoning:**
Multi-file processing requires determining which document is authoritative and reconciling conflicting fields across documents. This is a significant engineering complexity for an edge case that affects ~10% of contracts. Deferred to v2 with a clear workaround: ops can manually correct any fields that reflect outdated terms from an earlier version.

---

### Tradeoff 7 — CFO Dashboard as Separate View vs. Embedded in Ops Flow

**Chosen:** Separate view, separate URL, read-only for CFO

**Rejected:** Embed financial metrics in the ops activation screen

**Reasoning:**
The CFO's mental model is portfolio-level (all contracts, revenue trends, pipeline) while the ops user's mental model is task-level (this contract, right now). Mixing the two contexts creates noise for both users. Separate views serve each user's job-to-be-done without compromise. The CFO dashboard is also read-only — no ability to approve or modify contracts — maintaining clean separation of responsibilities.

---

## What We Deferred to v2

| Feature | Reason | v2 Approach |
|---|---|---|
| Full automation (no human review) | Requires accuracy validation at scale first | Enable after 1,000 contracts, accuracy > 99.5% |
| RAG plan catalogue validation | Engineering complexity for v1 | Vector DB of plan catalogue, LLM cross-references on extraction |
| MCP integration layer | Reduces custom API work but adds architecture complexity | Replace custom billing + messaging APIs with MCP protocol |
| DocuSign webhook trigger | Requires DocuSign partnership/integration | Webhook fires on contract signed → auto-uploads to activation engine |
| CRM sync (Salesforce / HubSpot) | Significant integration scope | Update opportunity stage to "Customer - Active" post-activation |
| Multi-language contract support | LLM accuracy lower on non-English contracts | Dedicated language-specific extraction models |
| Multi-product contract extraction | Complex parsing logic | Extract all line items, create multiple subscriptions |
| Mobile interface | Ops primarily on desktop | Progressive web app in v2 |
| Bulk contract processing | Edge case for v1 | Queue-based processing for batch uploads |
| Audit log export | Compliance requirement for enterprise tier | Full field-level change log, exportable to CSV |
| Amendment handling | Affects ~10% of contracts | Reconcile fields across base contract + amendments |

---

*Edge Cases & Tradeoffs Author: Abinaya J | Version 1.0 | February 2026*
