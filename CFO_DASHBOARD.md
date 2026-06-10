# CFO Dashboard Specification
## Revenue Intelligence Layer — Contract Activation Engine

---

## Why This Exists

Most ops tools optimise for the ops user. This dashboard exists for a different stakeholder entirely.

The CFO's question is not *"how fast are we activating contracts?"* It is:

> **"How much money are we leaving on the table every month because of activation delays — and what did we gain by fixing it?"**

No existing tool answers that question. This dashboard does.

---

## The Business Case in Numbers

For a B2B SaaS company with:
- 20 contracts signed per month
- Average contract value: $500,000 ARR ($41,667/month)
- Average activation delay: 3 business days

**Revenue delayed per month:**
20 contracts × $41,667/month ÷ 22 working days × 3 days = **$113,636 in delayed MRR**

**After activation engine:**
Average activation time: 90 seconds
Revenue delay: effectively zero

**Annual impact:** $113,636 × 12 = **$1.36M in previously delayed revenue, now recognised on time**

This is the number the CFO cares about. Everything else in this dashboard supports it.

---

## Dashboard Structure

### Panel 1 — Revenue Impact (Hero Panel)

**Purpose:** Show the CFO the financial value of the activation engine in dollar terms.

| Metric | Definition | Display |
|---|---|---|
| MRR Delay Eliminated | (Avg contract value / 22 days) × days saved × contracts/month | Large number, $ format |
| Annual Revenue Impact | MRR delay eliminated × 12 | Highlighted callout |
| Cash Flow: Before vs After | Side-by-side chart — revenue recognition timeline pre and post activation engine | Bar chart |
| Revenue at Risk (Live) | Contracts uploaded but not yet activated × value × days pending | Red indicator if > threshold |

**Cash Flow Chart:**

```
Revenue Recognition Timeline

Before:                  After:
Jan  ──────────────►    Jan  ──►
     Day 1          Day 4        Day 1 (90 sec)

     [Contract signed]           [Contract signed]
          ↓                           ↓
     [3 days manual]           [90 seconds AI]
          ↓                           ↓
     [Revenue starts]          [Revenue starts]
```

---

### Panel 2 — Ops Efficiency

**Purpose:** Show the productivity multiplier for the billing/ops team.

| Metric | Before | After | Display |
|---|---|---|---|
| Contracts processed/ops/day | 3–4 | 20+ | Side-by-side stat |
| Avg time per contract | 45 minutes | 90 seconds | Side-by-side stat |
| Ops hours saved per month | Baseline | Calculated | Trend line |
| Cost per activation | High (manual labour) | Low (AI) | Cost curve |

**Efficiency Multiplier Callout:**
> *"Your ops team is now processing 6× more contracts per day with the same headcount."*

---

### Panel 3 — Pipeline Visibility

**Purpose:** Real-time view of every contract in the activation pipeline — so the CFO can see what's moving and what's stuck.

| Stage | Count | Avg Time in Stage | Total Value |
|---|---|---|---|
| Uploaded — Awaiting Review | 3 | 2 hours | $1.2M |
| In Review | 1 | 15 minutes | $800K |
| Pending Finance Approval | 2 | 18 hours | $3.1M |
| Activated This Month | 18 | 52 seconds avg | $9.4M |

**Flags:**
- 🔴 Oldest unactivated contract: "Acme Corp — uploaded 3 days ago — $2.1M"
- 🟡 Finance approval pending > 24 hours: "2 contracts"

---

### Panel 4 — Quality & Accuracy

**Purpose:** Assure the CFO that speed has not come at the cost of accuracy.

| Metric | Value | Trend |
|---|---|---|
| AI extraction accuracy | 96.4% | ↑ |
| Billing corrections raised post-activation | 0 this month | ✅ |
| Fields requiring manual correction | 8.2% | ↓ |
| Manual fallback rate (AI failure) | 1.8% | ↓ |
| Duplicate subscription rate | 0% | ✅ |

**Key message:** *"We are faster AND more accurate than the manual baseline."*

---

### Panel 5 — Time Period Comparison

**Purpose:** Let the CFO compare performance before and after the activation engine was deployed.

- Toggle: Last 30 days / Last 90 days / Since Launch
- Key before/after benchmarks locked in at launch date
- Month-over-month trend for all primary metrics

---

## Design Principles for This Dashboard

**1. Lead with money, not operations**
The first number the CFO sees is revenue impact in dollars — not activation time in seconds. Translate everything to business value.

**2. Separate from the ops view**
Ops sees the contract queue and review form. CFO sees aggregated intelligence. Different user, different screen, different mental model.

**3. Real-time pipeline visibility**
The "revenue at risk" indicator updates live. A CFO checking this daily can immediately see if contracts are stuck — and ask why.

**4. Before vs. after is always visible**
The value of the product is in the delta. Every panel shows the before state as a benchmark.

---

## How This Differentiates the Product

Most B2B SaaS ops tools stop at the operations layer. Adding a CFO dashboard shifts the product from:

**"A tool that makes ops faster"**

to:

**"A revenue intelligence platform that makes delayed cash flow visible and eliminates it."**

That is a fundamentally different sale. The ops team buys the first product. The CFO funds the second one.

---

*CFO Dashboard Spec Author: Abinaya J | Version 1.0 | February 2026*
