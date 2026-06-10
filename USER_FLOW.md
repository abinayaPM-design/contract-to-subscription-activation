# User Flow
## Contract-to-Subscription Activation Engine

---

## Primary Flow — Happy Path (Ops User)

```
START
  │
  ▼
[Screen 1] Upload Screen
  │
  │  User: Drags and drops contract file
  │  Accepted: PDF · DOCX · JPG · PNG · Scanned PDF
  │  System: Format + size validation
  │  System: OCR pre-processing if scanned/image
  │  User: Clicks "Process Contract"
  │
  ▼
[System] AI Extraction (< 10 seconds)
  │
  │  Progress indicator shown: 
  │  "Reading document... Extracting fields... Validating..."
  │  LLM processes full document
  │  Returns 12 fields + confidence scores
  │  Business rules validated
  │
  ▼
[Screen 2] Review & Approval Form
  │
  │  ┌──────────────────────────────────────────────────┐
  │  │  Customer Name     Acme Corp              🟢 96% │
  │  │  Plan Name         Enterprise Annual      🟢 94% │
  │  │  Billing Amount    $24,000                🟢 91% │
  │  │  Billing Currency  USD                    🟢 99% │
  │  │  Billing Frequency Annual                 🟢 95% │
  │  │  Start Date        2026-02-01             🟡 78% │ ← Review
  │  │  End Date          2027-01-31             🟢 92% │
  │  │  Seats             50                     🟢 88% │
  │  │  Payment Terms     [empty]                ⚪ N/A  │ ← Must enter
  │  │  Auto-Renewal      Yes                    🟢 90% │
  │  │  Discount          10% Year 1             🟡 74% │ ← Review
  │  │  Special Terms     None                   🟢 85% │
  │  └──────────────────────────────────────────────────┘
  │
  │  Approve button: DISABLED (Payment Terms missing)
  │
  │  User: Clicks Payment Terms field → enters "Net 30"
  │  User: Confirms Start Date (reviews, looks correct)
  │  User: Confirms Discount (reviews, looks correct)
  │
  │  Approve button: ENABLED (all fields confirmed)
  │
  │  User: Clicks "Approve & Activate"
  │
  ▼
[System] Final Validation Pass
  │
  │  ✅ All fields present and confirmed
  │  ✅ Price within Enterprise Annual range
  │  ✅ Start date within 90-day backdating limit
  │  ✅ Customer ID found in billing system
  │  ✅ No duplicate subscription detected
  │
  ▼
[System] Subscription Creation
  │
  │  POST to billing system API
  │  Returns: SUB-2026-00472
  │
  ▼
[Screen 3] Confirmation
  │
  │  ✅ Subscription Activated
  │  Subscription ID: SUB-2026-00472
  │  Customer: Acme Corp
  │  Plan: Enterprise Annual · $24,000/year
  │  Activated: 2026-02-01
  │  Time taken: 52 seconds
  │
  │  [View Subscription] [Process Another Contract]
  │
  │  Background: Sales rep notified via email
  │             CFO dashboard updated
  │
  ▼
END
```

---

## Alternate Flow 1 — Scanned or Image Document

```
[Screen 1] Upload Screen
  │
  │  User uploads scanned PDF or image file
  │
  ▼
[System] OCR Pre-Processing
  │
  │  OCR converts image to text
  │  Quality check: is OCR output readable?
  │
  ├── High quality OCR → continue to AI extraction (normal flow)
  │
  └── Low quality OCR
        │
        ▼
      Warning banner on Screen 2:
      "Scanned document quality is low — 
       extraction confidence may be reduced.
       Review all fields carefully."
      
      Form shown with more yellow/red fields than usual
      Ops reviews more carefully → approves normally
```

---

## Alternate Flow 2 — Low Confidence Fields

```
[Screen 2] Review Form
  │
  │  One or more fields show 🔴 (confidence < 70%)
  │  Approve button DISABLED
  │
  ▼
User clicks red field
  │
  │  Field expands with:
  │  - Current extracted value (pre-filled)
  │  - "Why is this flagged?" tooltip
  │  - Editable input
  │
  ▼
User reviews → corrects if needed → clicks "Confirm Field"
  │
  │  Field badge changes from 🔴 to ✅ (manually confirmed)
  │
  ▼
All red/missing fields confirmed?
  │
  ├── No → Approve button stays disabled
  └── Yes → Approve button activates
              │
              ▼
           Normal approval flow continues
```

---

## Alternate Flow 3 — Finance Approval Required

```
[Screen 2] Review Form
  │
  │  Ops completes review normally
  │  Billing Amount = $180,000 (above $50,000 threshold)
  │
  ▼
Ops clicks "Submit for Approval"
  │
  ▼
[Screen 2b] Finance Review Pending
  │
  │  "This contract requires Finance Lead approval
  │   before activation. Submitted for review."
  │
  │  Status: Pending Finance Approval
  │  Submitted: Now
  │  Estimated SLA: 4 hours
  │
  ▼
[System] Finance Lead Notification
  │
  │  Email: "Contract pending approval — Acme Corp, $180,000"
  │  Link → same review form (read-only for finance)
  │
  ▼
Finance Lead reviews form
  │
  ├── Approves
  │     │
  │     ▼
  │   Subscription created automatically
  │   Both ops user and sales rep notified
  │
  └── Rejects (with comment)
        │
        ▼
      Ops notified: "[Finance Lead]: Price doesn't match
      approved quote. Please verify with sales."
      Contract returned to ops for correction
```

---

## Alternate Flow 4 — AI Extraction Fails

```
[System] AI Extraction
  │
  │  ❌ LLM API error / document unreadable
  │
  ▼
[Screen 2c] Manual Entry Fallback
  │
  │  Banner: "Automatic extraction unavailable.
  │           Please enter contract details manually."
  │
  │  Blank form shown — all 12 fields empty
  │  All fields editable
  │  No confidence badges shown
  │
  ▼
Ops fills in all required fields manually
  │
  ▼
Ops clicks "Submit for Approval"
  │
  ▼
Normal validation + approval flow continues
```

---

## Alternate Flow 5 — Duplicate Contract Detected

```
[System] Document Upload Validation
  │
  │  File hash matches contract processed 15 days ago
  │
  ▼
[Screen 1 — Warning Banner]
  │
  │  "⚠️ Similar contract detected
  │   A contract for Acme Corp was processed on Jan 15, 2026
  │   for $24,000 Annual. Subscription ID: SUB-2026-00318
  │
  │   Is this a renewal or a new agreement?"
  │
  │  [This is a new agreement — continue]
  │  [View existing subscription]
  │
  ├── User confirms new agreement → normal flow continues
  │
  └── User views existing subscription → workflow ends
```

---

## Alternate Flow 6 — Non-Contract Document Uploaded

```
[System] AI Extraction
  │
  │  Document processed — all fields return very low confidence
  │  (e.g. user uploaded an invoice or NDA by mistake)
  │
  ▼
[Screen 2 — All Fields Red + Banner]
  │
  │  "⚠️ This document doesn't appear to be a subscription contract.
  │   All fields require manual review."
  │
  │  Manual entry fallback offered:
  │  [Enter details manually] [Upload a different document]
```

---

## Screen Summary

| Screen | Name | Primary User Action |
|---|---|---|
| Screen 1 | Upload | Upload contract file |
| Screen 2 | Review & Approval | Review fields, edit, approve |
| Screen 2b | Finance Pending | Submit for finance review |
| Screen 2c | Manual Fallback | Enter fields manually |
| Screen 3 | Confirmation | View subscription ID |
| Screen 4 | CFO Dashboard | Review revenue + pipeline metrics |

---

*User Flow Author: Abinaya J | Version 1.0 | February 2026*
