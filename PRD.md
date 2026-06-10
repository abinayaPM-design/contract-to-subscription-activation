# Product Requirements Document
## Contract-to-Subscription Activation Engine

| Field | Detail |
|---|---|
| Author | Abinaya J |
| Status | v1.0 — Final |
| Date | February 2026 |
| Type | 0→1 Product Build |
| Priority | P0 |
| Reviewers | Engineering Lead, Finance Lead, Head of Ops |

---

## 1. Problem Statement

B2B SaaS companies lose 2–5 business days between contract signing and customer go-live due to manual data extraction and entry into billing systems.

**Impact:**
- **Revenue delay:** Subscription billing starts 2–5 days late per customer
- **Ops inefficiency:** Billing teams process 3–4 contracts/day max; capacity limited by reading time
- **Error risk:** Manual data entry introduces billing mistakes — wrong prices, wrong cycles, wrong dates
- **Customer experience:** Customers who just signed a contract wait days to use the product they paid for

**Root cause:** The bottleneck is not the approval — it is the data extraction and entry step. Automating this collapses the entire process.

---

## 2. Goals

### Must Achieve (v1)
- Reduce contract-to-subscription activation time to under 90 seconds (happy path)
- Maintain or improve billing data accuracy vs. manual baseline
- Require zero contract reading by the approver
- Accept any document type — PDF, DOCX, scanned documents, images

### Must Not
- Increase billing error rate vs. baseline
- Remove human accountability from the final approval step
- Break existing subscription creation workflows if the system fails
- Create duplicate subscriptions

### Business Goals
- Enable CFO visibility into revenue delay and activation pipeline
- Reduce ops headcount requirement for contract processing
- Create a defensible activation layer between CLM and billing systems

---

## 3. Non-Goals (Out of Scope for v1)

- E-signature or contract negotiation
- Contract version tracking or redlining
- Multi-party approval beyond a two-level flow (ops + finance)
- Automatic subscription creation with no human review
- Revenue recognition accounting entries
- Non-English contract support
- Mobile interface
- Bulk contract processing
- CRM sync (v2)
- DocuSign webhook trigger (v2)

---

## 4. Users

### Primary: Billing/Ops Specialist
**Frequency:** Daily
**Job:** Process signed contracts into active subscriptions without reading the whole document
**Success state:** Reviews a pre-filled form in 30 seconds, clicks Approve, subscription is live

### Secondary: Finance Lead
**Frequency:** Weekly (high-value contracts only)
**Job:** Approve contracts above value threshold before activation
**Success state:** Receives notification with pre-filled form, approves in under 4 hours

### Secondary: Sales Rep
**Frequency:** Per deal
**Job:** Know that customer will go live immediately after signing
**Success state:** Receives automatic notification when subscription is activated

### Tertiary: CFO
**Frequency:** Monthly
**Job:** Understand revenue delay and ops efficiency across activation pipeline
**Success state:** Dashboard shows cash flow impact, pipeline visibility, before vs. after comparison

---

## 5. User Stories

| ID | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| US-01 | Ops user | Upload any contract document | System extracts subscription fields automatically | P0 |
| US-02 | Ops user | See extracted fields in an editable form with confidence indicators | I review without reading the contract | P0 |
| US-03 | Ops user | See which fields need my attention (red flags) | I know where to focus my review | P0 |
| US-04 | Ops user | Edit any field before approving | I can correct AI errors before they reach billing | P0 |
| US-05 | Ops user | Approve with one click after review | Subscription gets created automatically | P0 |
| US-06 | Ops user | See clear confirmation with subscription ID | I know activation succeeded | P0 |
| US-07 | Ops user | Use a manual entry form if AI fails | Workflow never breaks | P0 |
| US-08 | Finance lead | Be notified for contracts above threshold | I can review before activation | P1 |
| US-09 | Finance lead | Approve or reject with a comment | Decision is logged for audit | P1 |
| US-10 | Sales rep | Receive notification when customer goes live | I can confirm go-live with the customer | P1 |
| US-11 | CFO | See cash flow before vs. after on a dashboard | I understand the financial impact | P1 |
| US-12 | CFO | See pipeline of contracts at each activation stage | I can identify bottlenecks | P1 |
| US-13 | Any user | Upload scanned PDFs and image files | Document format is never a barrier | P0 |

---

## 6. Fields Extracted by AI

| # | Field | Required | Example | Validation Rule |
|---|---|---|---|---|
| 1 | Customer Name | Yes | Acme Corp | Must match existing customer record in billing system |
| 2 | Plan Name | Yes | Enterprise Annual | Must match active plan in plan catalogue |
| 3 | Billing Amount | Yes | $24,000 | Must be within plan's min/max price range |
| 4 | Billing Currency | Yes | USD | ISO 4217 currency code |
| 5 | Billing Frequency | Yes | Annual | Monthly / Quarterly / Annual only |
| 6 | Contract Start Date | Yes | 2026-02-01 | Not more than 90 days in past |
| 7 | Contract End Date | Yes | 2027-01-31 | Must be after start date |
| 8 | Number of Seats | If applicable | 50 | Positive integer |
| 9 | Payment Terms | Yes | Net 30 | Net 15 / Net 30 / Net 45 / Upfront |
| 10 | Auto-Renewal | Yes | Yes | Boolean |
| 11 | Discount Applied | If applicable | 10% Year 1 | Validated against approved discount schedule |
| 12 | Special Terms | If applicable | Free onboarding included | Flagged for manual review — no auto-validation |

---

## 7. Confidence Scoring Rules

| Score | Label | Badge | Approve Button |
|---|---|---|---|
| 90–100% | High Confidence | 🟢 | Unaffected |
| 70–89% | Review Recommended | 🟡 | Unaffected |
| Below 70% | Verify Required | 🔴 | **Disabled** until field confirmed |
| Not extracted | Missing | ⚪ | **Disabled** until field entered |

**Rule:** Approve button is disabled if ANY field is below 70% confidence or missing. It activates only when all fields are either high/medium confidence, or manually confirmed by the ops user.

---

## 8. Document Type Handling

| Document Type | Processing | Confidence Impact | User Message |
|---|---|---|---|
| Text-based PDF | Direct LLM extraction | High baseline | None |
| DOCX | Direct LLM extraction | High baseline | None |
| Scanned PDF (image) | OCR → LLM | Slightly lower | "Scanned document detected — review carefully" |
| Image file (JPG/PNG) | OCR → LLM | Lower baseline | "Image document — extraction may be less accurate" |
| Password-protected PDF | Rejected | N/A | "Please upload an unlocked version" |
| Non-contract document | All fields low confidence | Very low | Manual fallback triggered automatically |
| Multi-page contract (50+ pages) | Full processing | Normal | "Large document — may take up to 30 seconds" |
| Contract with amendments | Single file processed | Normal | "If this includes amendments, confirm it is the final version" |

---

## 9. System Flow

```
[1] Document Upload
    ├── Format validation (PDF/DOCX/image/scan)
    ├── Size check (max 20MB)
    ├── Virus scan
    └── If scanned/image: OCR pre-processing
         ↓
[2] AI Extraction
    ├── LLM processes full document
    ├── Returns 12 fields + confidence scores
    ├── Processing target: < 10 seconds
    └── If extraction fails: trigger manual fallback
         ↓
[3] Business Rule Validation
    ├── Customer ID lookup
    ├── Plan name validation
    ├── Price range check
    ├── Date logic check
    └── Duplicate contract detection
         ↓
[4] Review Screen Rendered
    ├── Pre-filled form with confidence badges
    ├── Red fields: Approve button disabled
    ├── Anomaly flags shown inline
    └── Ops reviews, edits as needed
         ↓
[5] Approval Submitted
    ├── All fields green/yellow or manually confirmed
    ├── Final validation pass
    └── If contract value > threshold: route to finance
         ↓
[5a] Finance Approval (if triggered)
    ├── Email notification to finance lead
    ├── Finance reviews same form (read-only)
    └── Approves or rejects with comment
         ↓
[6] Subscription Creation
    ├── POST to billing system API
    ├── Returns subscription ID
    └── Confirmation screen shown
         ↓
[7] Post-Activation
    ├── Sales rep notification sent
    ├── Activation time logged
    └── CFO dashboard updated
```

---

## 10. Error States

| Scenario | System Behaviour | User Message |
|---|---|---|
| Invalid file type | Block upload | "Accepted formats: PDF, DOCX, JPG, PNG" |
| File too large (> 20MB) | Block upload | "File exceeds 20MB limit — compress or split the document" |
| AI extraction fails completely | Show blank manual entry form | "Automatic extraction unavailable — please enter details manually" |
| Customer not found in billing system | Block approval | "Customer not found — create customer record first" |
| Plan name not in catalogue | Flag field red | "Plan not recognised — select from available plans" |
| Price outside allowed range | Flag field red | "Amount outside allowed range for this plan" |
| Backdated start date > 90 days | Flag field red, require finance approval | "Start date is more than 90 days in the past — finance approval required" |
| Billing API timeout | Do not create subscription, show retry | "Activation failed — please retry. Your review data is saved." |
| Duplicate contract detected | Warning banner, allow override | "Similar contract processed on [date] — confirm this is a new agreement" |
| Two ops users reviewing simultaneously | Lock contract, show active reviewer | "In review by [name] — check back in a few minutes" |

---

## 11. RICE Prioritisation

| Feature | Reach | Impact | Confidence | Effort (weeks) | Score |
|---|---|---|---|---|---|
| Any document upload + AI extraction | High | High | High | 2 | 9.0 |
| Confidence scoring + field flagging | High | High | High | 1 | 9.5 |
| Editable review form | High | High | High | 1 | 9.5 |
| Approve + subscription creation | High | High | High | 2 | 9.0 |
| Manual entry fallback | High | High | High | 0.5 | 9.8 |
| CFO revenue dashboard | Medium | High | High | 2 | 7.5 |
| Finance approval workflow | Medium | Medium | High | 1.5 | 6.0 |
| Sales rep notification | Medium | Low | High | 0.5 | 5.0 |
| Duplicate detection | Low | High | High | 1 | 6.0 |
| Concurrent user locking | Low | Medium | High | 0.5 | 4.5 |
| Audit log | Low | Medium | High | 1 | 4.0 |
| Multi-language support | Low | Low | Medium | 4 | 1.0 |

**v1 scope:** Top 6 features (above the line at score 7.0+)

---

## 12. Open Questions

| # | Question | Owner | Priority | Status |
|---|---|---|---|---|
| 1 | What is the contract value threshold for finance approval? | Finance Lead | High | Open |
| 2 | Should sales reps upload directly, or ops only in v1? | Head of Ops | High | Open |
| 3 | What is the data retention policy for uploaded contracts? | Legal | High | Open |
| 4 | Do we need a full audit log of every field edit for compliance? | Compliance | Medium | Open |
| 5 | Which billing system do we integrate with first? | Engineering | High | Open |
| 6 | What is the acceptable finance approval SLA (4 hours / 24 hours)? | Finance Lead | Medium | Open |
| 7 | Can we use contract data for extraction model improvement? | Legal | Medium | Open |

---

*PRD Author: Abinaya J | Version 1.0 | February 2026*
