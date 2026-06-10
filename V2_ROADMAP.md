# v2 Roadmap
## RAG + MCP Architecture — Contract Activation Engine

---

## Why v2 Exists

v1 solves the core problem: contract uploaded → AI extracts → human reviews → subscription created in < 90 seconds.

v2 makes the AI smarter, the integrations seamless, and the human review step optional.

The two technologies that enable this are **RAG** (Retrieval Augmented Generation) and **MCP** (Model Context Protocol).

---

## v1 vs. v2 Architecture

### v1 Architecture (Current)
```
Contract PDF
      │
      ▼
LLM reads full document
      │
      ▼
Returns field values + confidence scores
(based only on what's in the contract)
      │
      ▼
Ops reviews → approves → subscription created via API
```

**Limitation:** The LLM has no knowledge of your business. It doesn't know what plans exist, what the valid price ranges are, or what billing configurations are allowed. It can only extract what it reads. Validation happens as a separate post-extraction step.

---

### v2 Architecture (Planned)

```
Contract PDF
      │
      ▼
LLM reads full document
      │
      ▼
RAG: Cross-reference extracted fields          ← NEW
against vector DB of plan catalogue,
pricing tiers, billing configurations
      │
      ▼
Returns field values + confidence scores        
(now informed by ground truth data)            ← SMARTER
      │
      ▼
Fewer red fields → faster ops review            ← BETTER UX
      │
      ▼
MCP: Create subscription + notify Slack         ← NEW
+ update CRM — all via one protocol
```

---

## RAG — Retrieval Augmented Generation

### What It Is

RAG is a technique where, instead of asking an LLM to answer from memory alone, you first retrieve relevant information from a knowledge base and include it in the prompt.

In our context: when the LLM extracts "Plan: Enterprise Plus, $24,000/year" from the contract, RAG retrieves the Enterprise Plus plan record from a vector database and includes it in the extraction prompt.

The LLM can then validate: *"The contract says $24,000. The plan record says $24,000 is a valid price for this plan. Confidence: 97%."*

Without RAG: *"The contract says $24,000. I have no way to verify if this is correct. Confidence: 78%."*

### What Goes in the Vector Database

| Knowledge Source | Content | Why It Helps |
|---|---|---|
| Plan catalogue | All active plans, prices, billing frequencies, seat limits | Validates plan name + price in one step |
| Pricing tiers | Min/max prices per plan, volume discounts, approved exceptions | Catches out-of-range prices before ops review |
| Customer records | Existing customer names, IDs, active subscriptions | Validates customer match, detects duplicates |
| Payment terms dictionary | Approved payment terms with definitions | Catches non-standard payment terms |
| Historical contracts | Anonymised field patterns from past contracts | Improves extraction on unusual contract formats |

### Impact on Accuracy

| Field | Without RAG | With RAG |
|---|---|---|
| Plan Name | 85–92% | 95–99% |
| Billing Amount | 88–94% | 96–99% |
| Billing Frequency | 90–95% | 97–99% |
| Payment Terms | 78–85% | 92–97% |
| Customer Name | 88–94% | 95–99% |

**Result:** Fewer red fields → less ops review time → closer to full automation.

### The Path to Full Automation

v2 RAG target: > 99.5% accuracy on all fields across 1,000+ contracts

Once this is validated, v3 introduces:
- Auto-approve if all fields above 99% confidence + RAG-validated
- Human review only for exceptions (red fields, high-value contracts, custom terms)
- Activation time: < 10 seconds

---

## MCP — Model Context Protocol

### What It Is

MCP is an open protocol (introduced by Anthropic, 2024) that standardises how AI models connect to external tools and data sources.

Without MCP, every integration requires custom API code:
- Custom billing system integration
- Custom Slack integration
- Custom Salesforce integration
- Custom Notion integration

Each integration has its own authentication, error handling, rate limits, and maintenance burden.

With MCP, the AI agent connects to any MCP-compatible server through one protocol. Add a new integration by pointing to a new MCP server — no new custom code.

### MCP Servers for v2

| MCP Server | What It Enables | Activation Trigger |
|---|---|---|
| Billing System | Create subscription, retrieve plan catalogue | Post-approval |
| Slack | Notify sales rep when customer goes live | Post-activation |
| Salesforce / HubSpot CRM | Update opportunity to "Customer - Active" | Post-activation |
| Notion | Retrieve plan documentation for RAG | At extraction time |
| E-signature platform | Receive webhook when contract signed → auto-upload | Contract signed event |

### The v2 Activation Flow with MCP

```
E-signature platform: Contract signed
      │
      ▼ (webhook via MCP)
Contract auto-uploaded to activation engine
      │
      ▼
LLM extracts fields
      │
      ▼ (MCP → Notion)
RAG retrieves plan catalogue from Notion
      │
      ▼
Fields validated + confidence scored
      │
      ▼
[If all fields > 99% confidence + RAG validated]
Auto-approve path:
      │
      ▼ (MCP → Billing system)
Subscription created automatically
      │
      ▼ (MCP → Slack)
Sales rep notified: "Acme Corp is live 🎉"
      │
      ▼ (MCP → Salesforce)
Opportunity updated: "Customer - Active"
      │
      ▼
CFO dashboard updated
      │
Total time: < 15 seconds. Zero human interaction.

[If any field below threshold]
Route to ops review → standard v1 flow
```

---

## v2 Feature Prioritisation

| Feature | Value | Effort | Priority |
|---|---|---|---|
| RAG plan catalogue validation | High — improves accuracy, reduces ops friction | Medium | P0 |
| MCP billing system integration | High — cleaner than custom API, extensible | Medium | P0 |
| E-signature platform webhook (auto-upload) | High — eliminates manual upload step | Low (MCP) | P1 |
| MCP Slack integration | Medium — nice UX for sales rep | Low (MCP) | P1 |
| CRM sync via MCP | Medium — ops hygiene | Low (MCP) | P1 |
| Auto-approve path (no human review) | Very High — full automation | High (requires accuracy validation) | P2 |
| Multi-language contract support | Medium — expands market | High | P2 |
| Multi-product contract extraction | Medium — covers complex deals | High | P2 |
| Audit log export | Medium — enterprise compliance | Medium | P2 |

---

## v2 Success Criteria

| Metric | v1 Target | v2 Target |
|---|---|---|
| Activation time P50 | < 90 seconds | < 15 seconds (auto path) |
| AI extraction accuracy | > 95% | > 99.5% |
| Fields requiring ops correction | < 10% | < 2% |
| Auto-approve rate (no human review) | 0% (not enabled) | > 70% of contracts |
| Integration maintenance overhead | High (custom APIs) | Low (MCP protocol) |

---

## Why This Matters for the Product Strategy

v2 completes the vision: a fully automated activation layer that sits between contract signing and subscription billing — with humans in the loop only for exceptions.

The competitive moat deepens:
- E-signature platform triggers the upload automatically
- RAG ensures extraction accuracy without human review
- MCP makes the product infinitely integrable
- The CFO sees real-time activation without any ops involvement

This is not a workflow tool. It is **infrastructure for revenue activation**.

---

*v2 Roadmap Author: Abinaya J | Version 1.0 | February 2026*
