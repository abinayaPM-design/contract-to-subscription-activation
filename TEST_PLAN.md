# Test Plan — ActivateFlow Prototype
## Contract Test Suite (13 documents)

All contracts are between fictional **NimbusStack Software Inc.** (provider) and fictional customers. Each file targets a specific edge case from `EDGE_CASES.md`. Use this table during testing and demo recordings.

---

## Happy Path

| # | File | What's Inside | Expected Prototype Behaviour |
|---|---|---|---|
| 01 | `contract_01_happy_path.pdf` | Acme Corp, Enterprise Annual, $24,000/yr, Net 30, all 12 fields explicit | All fields extracted green/high confidence. Approve enabled quickly. The clean baseline demo. |
| 02 | `contract_02_missing_payment_terms.pdf` | Same deal, but payment terms described vaguely ("standard AP practices") and discount only implied ("approximately ten percent… subject to confirmation") | **Matches your Lovable demo state exactly:** Payment Terms = NOT FOUND (grey, manual entry), Discount = low confidence (red, verify). Approve blocked until both confirmed. |

## Data Edge Cases

| # | File | What's Inside | Expected Prototype Behaviour |
|---|---|---|---|
| 03 | `contract_03_ramp_pricing.pdf` | 3-year ramp: $10K → $15K → $20K, no auto-renewal, Net 45 | Billing Amount should be flagged — single field can't hold a ramp. Demo the "Ramp Pricing Detected" flag + notes field. Auto-Renewal = No. |
| 04 | `contract_04_multi_currency.pdf` | INR 5,10,000/quarter ≈ USD $6,000; invoicing currency "to be confirmed" | Currency field should flag ambiguity — demo the currency-selection dropdown behaviour. |
| 05 | `contract_05_backdated_start.pdf` | Start date 1 Dec 2025 — more than 90 days before "today" | Start Date flagged red + finance-approval banner triggered (backdating > 90 days rule). |
| 06 | `contract_06_free_trial.pdf` | 30-day free trial; Contract Start (1 Aug) ≠ Billing Start (31 Aug) | Demo the split contract_start vs billing_start handling. Monthly billing, Upfront terms. |
| 07 | `contract_07_usage_based.pdf` | $14/MAU, $2,500 monthly minimum, **no fixed annual fee** | Billing Amount flagged "Usage-Based" → manual entry for that field only. |

## Document Format Edge Cases

| # | File | What's Inside | Expected Prototype Behaviour |
|---|---|---|---|
| 08 | `contract_08_scanned.pdf` | Image-only PDF with scan artefacts (rotation, noise, blur). No text layer — try selecting text, you can't | OCR path: yellow "Scanned document detected" banner, lower confidence scores across the board. This is your **"Try: Scanned PDF"** demo chip scenario. |
| 09 | `contract_09_invoice_not_contract.pdf` | A tax invoice, not a contract at all | Non-contract detection: all fields very low confidence → banner "This doesn't appear to be a subscription contract" → manual-entry fallback offered. |
| 10 | `contract_10_with_amendment.pdf` | Page 1: contract at $60,000/200 seats. Page 2: Amendment No. 1 reducing to **$54,000/180 seats** | The authoritative-version problem. Correct extraction should take amendment values; a naive one takes page 1. Great interview talking point either way — demo the "confirm final version" warning. |
| 11 | `contract_11_password_protected.pdf` | Encrypted copy of contract 01. **Password: `secret123`** | Upload should be **rejected** with "Please upload an unlocked version." Never processed. |
| 12 | `contract_12_docx_format.docx` | Clean contract in DOCX — Business Annual, $18,500, Net 15, **Auto-Renewal: No**, 15% education discount full term | Tests DOCX ingestion path. All fields should extract cleanly. |
| 13 | `contract_13_german.pdf` | Complete contract in German — EUR 21.000, 30 Tage netto, 40 seats | Non-English detection: warn ops, flag fields red, offer manual entry (per v1 scope: non-English unsupported). |

---

## Suggested Demo Script (3 minutes, screen-share)

1. **Upload contract 01** → everything green → approve in ~40s → "this is the happy path, 90-second target"
2. **Upload contract 02** → two flagged fields block Approve → confirm both → "this is Layer 2 of the accuracy model — no low-confidence data reaches billing"
3. **Upload contract 08 (scanned)** → OCR banner + degraded confidence → "the system surfaces uncertainty instead of hiding it"
4. **Upload contract 11 (password)** → rejection → "and it fails gracefully"

That sequence demonstrates the accuracy framework, the edge-case handling, and the graceful degradation — the three pillars of the case study — in one sitting.

---

## Ground-Truth Answer Key (for measuring extraction accuracy)

If you later wire real AI extraction into the prototype, score it against these:

| File | Customer | Plan | Amount | Currency | Frequency | Start | End | Seats | Terms | Auto-Renew | Discount |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 01 | Acme Corp | Enterprise Annual | $24,000 | USD | Annual | 01-Feb-2026 | 31-Jan-2027 | 50 | Net 30 | Yes | 10% Yr 1 |
| 02 | Acme Corp | Enterprise Annual | $24,000 | USD | Annual | 01-Feb-2026 | 31-Jan-2027 | 50 | *(absent)* | Yes | ~10% (unconfirmed) |
| 03 | Brightline Logistics | Growth Multi-Year Ramp | $10K/$15K/$20K ramp | USD | Annual | 01-Mar-2026 | 28-Feb-2029 | 120 | Net 45 | No | None |
| 04 | Deccan Retail | Business Quarterly | INR 5,10,000 (~$6,000) | INR/USD ambiguous | Quarterly | 15-Jul-2026 | 14-Jul-2027 | 35 | Net 15 | Yes | None |
| 05 | Helios Energy | Professional Annual | $48,000 | USD | Annual | 01-Dec-2025 ⚠ | 30-Nov-2026 | 80 | Net 30 | Yes | None |
| 06 | Kite & Co | Starter Monthly | $900/mo | USD | Monthly | 01-Aug-2026 (billing 31-Aug) | 31-Jul-2027 | 10 | Upfront | Yes | None (30d trial) |
| 07 | Meridian Health | Scale Usage-Based | $14/MAU, $2.5K min ⚠ | USD | Monthly arrears | 01-Sep-2026 | 31-Aug-2027 | Variable | Net 30 | Yes | None |
| 08 | Ironwood Mfg | Professional Annual | $36,000 | USD | Annual | 01-Nov-2026 | 31-Oct-2027 | 65 | Net 30 | Yes | 5% Yr 1 |
| 10 | Vega Capital | Enterprise Annual | **$54,000** (amended) | USD | Annual | 01-Oct-2026 | 30-Sep-2027 | **180** (amended) | Net 30 | Yes | 10% reduction |
| 12 | Pinewood Education | Business Annual | $18,500 | USD | Annual | 10-Sep-2026 | 09-Sep-2027 | 25 | Net 15 | **No** | 15% full term |
| 13 | Müller & Hartmann | Enterprise Jährlich | EUR 21,000 | EUR | Annual | 01-Oct-2026 | 30-Sep-2027 | 40 | Net 30 | Yes | None |

*(09 and 11 have no answer key — they should never reach extraction.)*
