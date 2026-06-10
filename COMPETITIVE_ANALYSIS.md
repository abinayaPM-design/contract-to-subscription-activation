# Competitive Analysis
## Contract-to-Subscription Activation Engine

---

## The Market Landscape

The contract-to-subscription workflow touches three separate software categories — and none of them own the full journey.

---

## Category 1: Contract Lifecycle Management (CLM)

### DocuSign
- **What it does:** E-signature, contract storage, basic workflow routing
- **Strengths:** Market leader, ubiquitous, deep CRM integrations
- **Where it stops:** Delivers a signed PDF. What happens next is entirely manual.
- **Pricing:** $25–$300/user/month
- **Verdict:** Solves the signing problem. Creates the activation problem.

### Ironclad
- **What it does:** Contract creation, redlining, negotiation, approval workflows, contract repository
- **Strengths:** Strong for legal teams, excellent audit trail, Salesforce integration
- **Where it stops:** Executes the contract. Has no concept of what a "subscription" is.
- **Pricing:** $500–$2,000+/month (enterprise)
- **Verdict:** Best-in-class CLM. Stops the moment the contract is signed.

### Conga (formerly Apttus)
- **What it does:** Contract generation from templates, document assembly, CPQ integration
- **Strengths:** Deep Salesforce integration, good for high-volume contract generation
- **Where it stops:** Generates and stores contracts. No extraction or billing integration.
- **Pricing:** Custom enterprise pricing
- **Verdict:** Solves the upstream problem (creating contracts). Same downstream gap.

---

## Category 2: Subscription Billing

### Chargebee
- **What it does:** Subscription management, recurring billing, revenue recognition, dunning
- **Strengths:** Excellent API, flexible billing models, strong analytics
- **Where it stops:** Requires a human (or custom integration) to create the subscription from contract data
- **Pricing:** Free to $599+/month
- **Verdict:** Brilliant once the subscription exists. Can't create it from a contract.

### Stripe Billing
- **What it does:** Subscription billing, invoicing, payment processing
- **Strengths:** Developer-first, excellent APIs, global payments
- **Where it stops:** Same as Chargebee — data input is a human problem
- **Pricing:** 0.5–0.8% of billing volume
- **Verdict:** Infrastructure layer. Needs contract data fed to it manually.

### Recurly
- **What it does:** Subscription management, revenue recovery, analytics
- **Strengths:** Strong churn reduction tools, good for high-volume B2C
- **Where it stops:** Same upstream gap as all billing tools
- **Verdict:** Not differentiated from Chargebee for this use case.

---

## Category 3: Configure-Price-Quote (CPQ)

### Salesforce CPQ
- **What it does:** Guided selling, pricing configuration, quote generation, approval workflows
- **Strengths:** Deep Salesforce integration, handles complex pricing models
- **Where it stops:** Generates quotes and sometimes contracts — but subscription creation requires a separate step
- **Pricing:** $75/user/month+
- **Verdict:** Closest to bridging the gap, but still requires manual subscription creation.

### HubSpot Quotes
- **What it does:** Basic quoting within HubSpot CRM
- **Strengths:** Simple, free for HubSpot users
- **Where it stops:** Generates a quote document — no contract extraction or billing integration
- **Verdict:** SMB tool, not relevant for complex B2B contracts.

---

## The Gap Visualised

```
DEAL CLOSED
     │
     ▼
┌──────────────────────────────────────────────────────────────────┐
│  CONTRACT MANAGEMENT ZONE                                        │
│  DocuSign · Ironclad · Conga · Salesforce CPQ                   │
│                                                                  │
│  Generate → Negotiate → Sign → Store                            │
│                                              STOPS HERE ──────► │
└──────────────────────────────────────────────────────────────────┘
                                                      │
                                    ╔═════════════════▼═══════════╗
                                    ║   THE UNOWNED GAP           ║
                                    ║   48–120 hours              ║
                                    ║   Manual extraction         ║
                                    ║   Manual data entry         ║
                                    ║   Human error risk          ║
                                    ║                             ║
                                    ║   THIS IS THE PRODUCT       ║
                                    ╚═════════════════════════════╝
                                                      │
┌──────────────────────────────────────────────────────────────────┐
│  BILLING ZONE                                                    │
│  Chargebee · Stripe · Recurly                                    │
│                                                                  │
│  ► STARTS HERE  Create → Invoice → Collect → Renew              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Why Nobody Has Built This Yet

**1. It requires reading unstructured documents**
Contracts are not standardised. Every company uses a different template, different field names, different structures. Before LLMs, reliable extraction was impossible without expensive custom OCR + rule-based parsers per contract type.

**2. It requires crossing two product categories**
CLM vendors don't want to build a billing integration. Billing vendors don't want to build a document reader. The gap exists precisely because it falls between two product domains.

**3. The cost of errors is high**
Billing errors cause revenue disputes, customer churn, and legal liability. Companies defaulted to humans because the risk of automation errors was too high. LLMs with confidence scoring and human-in-the-loop review change this calculus.

**4. The market is large enough to ignore**
Enterprise companies built custom CRM-to-billing integrations in-house. The mid-market ($5M–$50M ARR SaaS) doesn't have the engineering resources for that — and has had no alternative. That is the target market.

**Market sizing (rough TAM):**
- ~30,000 B2B SaaS companies globally in the $5M–$50M ARR band
- Each processing 10–50 contracts/month with 0.5–2 FTE on billing ops
- At $500–$2,000/month per customer (priced against ops time saved + revenue delay eliminated): **$180M–$720M annual TAM** for the activation layer alone — before expanding into adjacent contract intelligence
- Bottom-up sanity check: capturing 2% of this market = 600 customers × $12K ACV = $7.2M ARR

---

## Our Differentiation

| Capability | DocuSign | Ironclad | Chargebee | Us |
|---|---|---|---|---|
| Contract signing | ✅ | ✅ | ❌ | ❌ |
| Contract storage | ✅ | ✅ | ❌ | ❌ |
| AI field extraction | ❌ | Limited | ❌ | ✅ |
| Confidence scoring | ❌ | ❌ | ❌ | ✅ |
| Human review layer | ❌ | ✅ (legal) | ❌ | ✅ (billing) |
| Subscription creation | ❌ | ❌ | Manual | ✅ Auto |
| CFO revenue dashboard | ❌ | ❌ | Partial | ✅ |
| Any document type | ❌ | ❌ | ❌ | ✅ |
| Activation time | Days | Days | Manual | < 90 sec |

---

## The Partnership Angle (v2+)

Rather than competing with DocuSign or Chargebee, the strongest go-to-market is **integrating with both**:

- **DocuSign webhook** → triggers contract upload automatically when contract is signed
- **Chargebee API** → creates subscription upon approval
- **Salesforce** → updates opportunity stage to "Customer - Active" post-activation

This positions the product as the **activation middleware layer** — not a replacement, but the missing piece that makes existing tools complete.

---

*Competitive Analysis Author: Abinaya J | Version 1.0 | February 2026*
