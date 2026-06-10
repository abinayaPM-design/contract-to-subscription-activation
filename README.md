# Contract-to-Subscription Activation Engine
### B2B SaaS Billing Platform — Senior PM Case Study

> **Role:** Product Lead (Solo)
> **Timeline:** February 2026
> **Type:** 0→1 Product Build
> **Prototype:** [Live on Lovable →](#) *(add your link)*
> **Status:** Prototype Live · v2 Roadmap Defined

---

## The One-Line Problem

> **Every B2B SaaS company loses 2–5 business days between a signed contract and a live customer — not because of approval delays, but because of manual data entry.**

That gap is the product.

---

## Table of Contents

1. [Problem Discovery](#problem-discovery)
2. [Market Context & Competitive Gap](#market-context--competitive-gap)
3. [Users & Jobs to Be Done](#users--jobs-to-be-done)
4. [Solution Overview](#solution-overview)
5. [The Accuracy Story](#the-accuracy-story)
6. [The CFO Layer — Revenue Intelligence](#the-cfo-layer--revenue-intelligence)
7. [What I Built](#what-i-built)
8. [Key Decisions & Tradeoffs](#key-decisions--tradeoffs)
9. [North Star & Metrics](#north-star--metrics)
10. [v2 Roadmap — RAG + MCP Architecture](#v2-roadmap--rag--mcp-architecture)

**Full Docs:**
- [PRD →](./docs/PRD.md)
- [User Flow →](./docs/USER_FLOW.md)
- [Accuracy Framework →](./docs/ACCURACY_FRAMEWORK.md)
- [Competitive Analysis →](./docs/COMPETITIVE_ANALYSIS.md)
- [CFO Dashboard Spec →](./docs/CFO_DASHBOARD.md)
- [Metrics & Instrumentation →](./docs/METRICS.md)
- [Edge Cases & Tradeoffs →](./docs/EDGE_CASES.md)
- [v2 Roadmap →](./docs/V2_ROADMAP.md)

---

## Problem Discovery

This problem was surfaced as an interview brief from a B2B SaaS billing platform.

The prompt: *"Automate and reduce the time from contract signing to subscription activation with high accuracy."*

**A note on method:** As an external case study, the workflow mapping below is based on secondary research and published ops process documentation rather than primary user interviews. In a real engagement, my first step would be 5–7 interviews with billing ops specialists to validate (or break) these assumptions.

Rather than jumping to a solution, I started with first principles:

**Why does this delay exist?**

After mapping the current workflow, the bottleneck is clear:

```
Contract Signed
      ↓
Ops receives PDF via email          ← No system handoff
      ↓
Ops reads entire contract           ← 30–60 minutes
      ↓
Ops manually enters fields          ← Typos, fatigue errors
into billing system
      ↓
Finance reviews (for large deals)   ← 1–2 days
      ↓
Ops creates subscription manually   ← Another handoff
      ↓
Customer goes live                  ← 2–5 business days later
```

**The insight:** The delay is not in the approval. It is in the data extraction and entry. Automate that, and the entire process collapses.

**The second insight:** Every existing tool in the market stops at the contract. None of them cross the gap into billing activation. That gap is unowned — and that is the product opportunity.

---

## Market Context & Competitive Gap

### The Existing Landscape

| Tool | Category | Where They Play | Where They Stop |
|---|---|---|---|
| DocuSign | E-signature | Contract signing, storage | Stops at signed PDF — hands off to human |
| Ironclad | Contract Lifecycle Management | Redlining, negotiation, approval workflows | Stops at executed contract |
| Conga | Contract Generation | Templates, document assembly | Stops at document creation |
| Chargebee / Stripe | Subscription Billing | Invoicing, recurring billing | Requires manual data input to start |
| Salesforce CPQ | Configure-Price-Quote | Quoting, pricing | Stops at quote approval |

### The Gap Nobody Owns

```
[Contract Signed] ──────────────────────────── [Subscription Live]
       ↑                    ↑                          ↑
   DocuSign           THIS IS THE GAP            Chargebee/Stripe
   Ironclad           (48–120 hours)
   Conga              Nobody owns this
```

Every tool above either manages the contract or manages the billing. None of them own the **activation layer** — the step that turns a signed contract into a live subscription.

### Why Nobody Has Solved This

1. **It requires reading contracts** — unstructured, variable, full of edge cases
2. **It requires connecting two systems** — contract storage and billing — that have historically been separate
3. **The cost of errors is high** — wrong billing data causes revenue disputes, churn, and legal issues — so companies defaulted to humans

**What changed:** LLMs can now read and extract from unstructured documents with high reliability when paired with validation layers. The blocker is gone. The product is now buildable.

### Our Positioning

> *"We are not a contract tool. We are not a billing tool. We are the activation layer between them."*

---

## Users & Jobs to Be Done

| User | Frequency | Job to Be Done | Current Pain |
|---|---|---|---|
| Billing/Ops Specialist | Daily | Process signed contract → activate subscription without reading the whole document | 30–60 min per contract, manual entry errors |
| Sales Rep | Per deal | Know customer will go live immediately after signing | No visibility, chases ops manually |
| Finance Lead | Weekly | Approve high-value contracts before activation | Reviews full contracts instead of key fields |
| CFO | Monthly | Understand revenue delay and ops efficiency across the pipeline | No visibility into activation bottleneck or its cost |

### Primary User: Billing/Ops Specialist

This is the user who owns the bottleneck. Solving for them solves the whole problem.

**Before:** Read 10-page contract → extract data manually → type into billing system → 45 minutes per contract → 3–4 contracts/day max

**After:** Upload contract → review pre-filled form → click Approve → 90 seconds → 20+ contracts/day

---

## Solution Overview

A three-screen activation flow with an intelligence layer underneath:

```
┌─────────────────────────────────────────────────────────┐
│                    UPLOAD CONTRACT                       │
│         PDF · DOCX · Scanned PDF · Image                │
│              Any document type accepted                  │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│                  AI EXTRACTION ENGINE                    │
│   Reads contract → extracts 12 fields → scores each     │
│   confidence 0–100% → validates against business rules  │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│               OPS REVIEW & APPROVAL                     │
│   Pre-filled editable form · Confidence indicators      │
│   Low-confidence fields flagged · Approve button        │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│              SUBSCRIPTION CREATED                        │
│   Auto-created via billing API · Confirmation shown     │
│   Sales rep notified · Activation time logged           │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│              CFO INTELLIGENCE LAYER                      │
│   Revenue delay eliminated · Ops efficiency gained      │
│   Cash flow before vs after · Pipeline visibility       │
└─────────────────────────────────────────────────────────┘
```

---

## The Accuracy Story

The biggest risk in automating billing data is a wrong value flowing into an invoice. We solve this with a **three-layer accuracy model** — not just AI, but AI + human review + validation rules.

### Layer 1 — AI Extraction with Confidence Scoring

Every field extracted gets a confidence score:

| Score | Label | UI | Action |
|---|---|---|---|
| 90–100% | High | 🟢 Green | Pre-filled, no action needed |
| 70–89% | Medium | 🟡 Yellow | Recommended review |
| Below 70% | Low | 🔴 Red | Mandatory review — Approve blocked |
| Not found | Missing | ⚪ Grey | Manual entry required |

### Layer 2 — Human-in-the-Loop Review

Ops never reads the contract. They review a 10-field form in 30 seconds. Any red field must be confirmed before approval. Human accountability is preserved at zero cost to speed.

### Layer 3 — Pre-Submission Validation

Before subscription creation, the system validates price range, plan existence, date logic, customer ID, and duplicate detection. If any check fails, the subscription is not created.

**Result:** Higher accuracy than manual entry, at 40× the speed.

*See full framework → [ACCURACY_FRAMEWORK.md](./docs/ACCURACY_FRAMEWORK.md)*

---

## The CFO Layer — Revenue Intelligence

This is the feature that separates an ops tool from a business intelligence product.

### The CFO's Problem

CFOs have no visibility into revenue delay. They know ARR. They don't know how much of that ARR is sitting in a signed contract waiting to be activated — and what that delay costs.

### What the Dashboard Shows

**Revenue Impact Panel:**
- Revenue delay eliminated (days saved × avg contract value × contracts/month)
- Revenue at risk (contracts uploaded but not yet activated × value × days pending)
- Monthly cash flow: before activation engine vs. after

**Ops Efficiency Panel:**
- Contracts processed per ops person per day (before: 3–4, after: 20+)
- Average activation time trend (week over week)
- Time saved per month across the team

**Quality Panel:**
- AI extraction accuracy rate
- Fields requiring manual correction (%)
- Billing error rate vs. baseline

**Pipeline Visibility:**
- Contracts by stage: Uploaded → In Review → Pending Finance → Activated
- Average time stuck at each stage
- Oldest unactivated contract flag

### Why This Matters for the CFO

> *"For a company doing $10M ARR with 20 contracts/month at an average of $500K each — a 3-day activation delay represents $82,000 in delayed monthly recurring revenue. The activation engine eliminates that entirely."*

*See full spec → [CFO_DASHBOARD.md](./docs/CFO_DASHBOARD.md)*

---

## What I Built

A working frontend prototype built on Lovable covering the complete happy path:

**Screen 1 — Upload**
- Drag and drop contract upload
- Accepts PDF, DOCX, scanned PDFs, and image files
- Processing animation with stage indicators

**Screen 2 — Review & Approval (the core screen)**
- 12 pre-filled fields with confidence badges (🟢🟡🔴)
- 2 fields pre-set to red to demonstrate the accuracy layer
- Editable form — click any field to correct
- Approve button disabled until all red fields confirmed
- Anomaly flags shown inline (e.g. backdated start date)

**Screen 3 — Confirmation**
- Subscription ID generated
- Activation timestamp
- Time taken displayed ("Activated in 52 seconds")
- Sales rep notification shown

**Screen 4 — CFO Dashboard**
- Cash flow before vs. after chart
- Ops efficiency metrics
- Pipeline stage breakdown
- Revenue at risk indicator

**Demo:** [Lovable Prototype →](#) *(add your link)*

---

## Key Decisions & Tradeoffs

| Decision | Chosen | Rejected | Why |
|---|---|---|---|
| Always require human review | ✅ Human approves every contract | Full automation if confidence > 95% | Billing errors compound — 5% error rate at scale = unacceptable |
| Confidence threshold for blocking | 70% | 50% or 80% | 70% is the inflection where errors become likely; 80% creates too much ops friction |
| Who can upload | Anyone (sales + ops) | Ops only | Sales reps are fastest to upload post-signing; accountability preserved at approval step |
| Fallback on AI failure | Blank manual entry form | Reject and ask to retry | Workflow never breaks — graceful degradation |
| Finance approval trigger | Configurable threshold ($50K default) | All contracts or none | Risk-proportionate; low-value contracts don't need finance review |
| CFO dashboard | Separate view, read-only | Embedded in ops flow | Different user, different job — don't mix contexts |

*See full tradeoff reasoning → [EDGE_CASES.md](./docs/EDGE_CASES.md)*

---

## North Star & Metrics

**North Star:** Time from contract upload to subscription live
**Target:** < 90 seconds (happy path)

| Layer | Metric | Target |
|---|---|---|
| Primary | Activation time P50 | < 90 seconds |
| Primary | Activation time P95 | < 5 minutes |
| Secondary | AI extraction accuracy | > 95% |
| Secondary | Fields requiring correction | < 10% |
| Secondary | Form completion rate | > 90% |
| Guardrail | Billing error rate | 0% increase vs. baseline |
| Guardrail | Duplicate subscription rate | < 0.1% |
| Business | Revenue delay eliminated / month | Track and report to CFO |

*Full instrumentation plan → [METRICS.md](./docs/METRICS.md)*

---

## v2 Roadmap — RAG + MCP Architecture

v1 uses a one-shot LLM extraction. v2 introduces a smarter architecture:

**RAG (Retrieval Augmented Generation):**
The LLM cross-references extracted fields against a vector database of your plan catalogue, pricing tiers, and billing configurations. When the contract says "Enterprise Plus, $24,000/year" — the system retrieves the exact plan record and validates the price. Confidence scores go up. Errors go down.

**MCP (Model Context Protocol):**
Instead of custom API integrations per tool, MCP lets the agent connect to your billing system, Notion (plan docs), Slack (sales notifications), and CRM — through one protocol. Faster to build, easier to maintain, infinitely extensible.

**The v2 flow:**
```
Contract uploaded
      ↓
LLM extracts fields
      ↓
RAG validates against plan catalogue    ← New
      ↓
Ops reviews (fewer red fields)          ← Better UX
      ↓
MCP creates subscription + notifies     ← New
sales rep on Slack + logs to CRM
```

*Full v2 spec → [V2_ROADMAP.md](./docs/V2_ROADMAP.md)*

---

## What I'd Do Next (If This Were a Real Company)

1. **Validate accuracy at scale** — run 100 real contracts through the extraction engine, measure field-level accuracy, identify systematic errors
2. **Instrument the review form** — which fields get edited most? Those are extraction improvement targets
3. **Ship the CFO dashboard to one customer** — does it change how they think about ops investment?
4. **Define the v2 RAG threshold** — at what accuracy level do we remove the human review step entirely?
5. **Explore the contract management adjacency** — once we own activation, do we expand left into contract storage?

---

*Built by Abinaya J — [linkedin.com/in/j-abinaya](https://linkedin.com/in/j-abinaya)*
*Case study documents the product thinking behind a working prototype. All company references are generic.*
