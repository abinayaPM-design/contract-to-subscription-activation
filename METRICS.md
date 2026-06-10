# Metrics & Instrumentation
## Contract-to-Subscription Activation Engine

---

## North Star Metric

> **Time from contract upload to subscription live**
> Target: < 90 seconds (P50 happy path)

This metric captures the entire value proposition in a single number. Everything else either explains it or protects it.

---

## Metric Hierarchy

### Primary Metrics — Did We Solve the Problem?

| Metric | Definition | Target | Why It Matters |
|---|---|---|---|
| Activation Time P50 | Median time upload → subscription live | < 90 seconds | Core value prop |
| Activation Time P95 | 95th percentile activation time | < 5 minutes | Catches outlier friction |
| Manual Baseline (locked) | Avg time before product launched | 2–5 business days | The comparison point |
| Activation Completion Rate | % of uploaded contracts that reach subscription created | > 90% | End-to-end funnel health |

---

### Secondary Metrics — Is the Product Working Well?

| Metric | Definition | Target |
|---|---|---|
| AI Extraction Accuracy | % of required fields extracted correctly (no edit needed) | > 95% |
| Fields Requiring Correction | % of extracted fields that ops edits before approving | < 10% |
| Review Screen Time | Avg time ops spends on the review form | < 60 seconds |
| Finance Approval SLA | Avg time for finance lead to approve high-value contracts | < 4 hours |
| Manual Fallback Rate | % of uploads that trigger manual entry fallback | < 3% |
| False Positive Rate | % of flagged fields that ops confirms without editing | < 15% |

---

### Guardrail Metrics — Are We Causing Harm?

| Metric | Definition | Target |
|---|---|---|
| Billing Error Rate | % of activated subscriptions with incorrect billing data | 0% increase vs. baseline |
| Duplicate Subscription Rate | % of activations creating a duplicate | < 0.1% |
| Subscription Creation Failure Rate | % of approved contracts where API call fails | < 0.5% |
| Ops Escalation Rate | % of contracts requiring out-of-flow intervention | < 5% |

---

### Business Metrics — CFO Layer

| Metric | Definition | Cadence |
|---|---|---|
| MRR Delay Eliminated | (Avg contract value / 22 days) × days saved × contracts activated | Monthly |
| Revenue at Risk | Unactivated contracts × value × days pending | Real-time |
| Ops Capacity Multiplier | Contracts processed per ops/day (before vs. after) | Monthly |
| Cost per Activation | Total ops cost allocated to activations / contracts processed | Monthly |

---

## Activation Funnel

```
Contract Uploaded                          100%
        │
        ▼
Extraction Completed                        97%  (3% → manual fallback)
        │
        ▼
Review Form Opened by Ops                   95%
        │
        ▼
Form Submitted for Approval                 91%
        │
        ├── Requires Finance Approval       15% of above
        │         │
        │         ▼
        │   Finance Approves                90% within SLA
        │
        ▼
Subscription Created Successfully           99%
        │
        ▼
Sales Rep Notified                         100% (post-creation)
```

**Where to investigate if funnel drops:**

| Drop Point | Likely Cause | Investigate |
|---|---|---|
| Extraction < 95% | Bad document quality or AI failure | Fallback rate, document types uploaded |
| Review form opened < 95% | Ops abandoning after seeing too many red fields | Avg red fields per form |
| Submission < 91% | Form too complex, too many mandatory corrections | Time on review screen, fields edited per session |
| Finance approval < 90% | Finance SLA not being met | Finance approval time distribution |
| Subscription created < 99% | Billing API issues | API error codes, retry success rate |

---

## Mixpanel / Amplitude Instrumentation

### Upload Events

| Event | Trigger | Key Properties |
|---|---|---|
| `contract_upload_started` | User selects file | file_type, file_size_mb |
| `contract_upload_completed` | File accepted and sent for processing | file_type, file_size_mb, is_scanned |
| `contract_upload_failed` | File rejected | failure_reason (size/format/virus) |

### Extraction Events

| Event | Trigger | Key Properties |
|---|---|---|
| `extraction_started` | AI begins processing | file_type, is_scanned |
| `extraction_completed` | AI returns results | extraction_time_ms, avg_confidence_score, num_fields_red, num_fields_yellow, num_fields_green, num_fields_missing |
| `extraction_failed` | AI returns error | failure_reason, fallback_triggered |
| `ocr_preprocessing_triggered` | Scanned/image document detected | ocr_quality_score |

### Review Events

| Event | Trigger | Key Properties |
|---|---|---|
| `review_form_opened` | Ops lands on review screen | num_red_fields, num_yellow_fields, num_missing_fields |
| `field_viewed` | Ops clicks on a field | field_name, confidence_score, is_red |
| `field_edited` | Ops changes a field value | field_name, original_value, new_value, confidence_score |
| `field_confirmed` | Ops confirms a red field without editing | field_name, confidence_score |
| `approve_button_enabled` | All blocking fields resolved | time_to_enable_ms, total_fields_edited |
| `review_abandoned` | Ops leaves without submitting | time_on_screen, num_fields_completed |

### Approval Events

| Event | Trigger | Key Properties |
|---|---|---|
| `approval_submitted` | Ops clicks Approve | time_on_review_screen_ms, num_fields_edited, num_fields_confirmed |
| `finance_review_triggered` | Contract above threshold | contract_value, threshold_value |
| `finance_approved` | Finance lead approves | time_to_approve_ms, finance_user_id |
| `finance_rejected` | Finance lead rejects | rejection_comment_length (not content) |

### Activation Events

| Event | Trigger | Key Properties |
|---|---|---|
| `subscription_creation_started` | Approval confirmed, API call initiated | contract_value, plan_name |
| `subscription_created` | Billing API returns success | subscription_id, plan_name, billing_amount, activation_time_ms |
| `subscription_creation_failed` | Billing API returns error | error_code, error_message, retry_count |
| `sales_rep_notified` | Notification sent post-activation | notification_channel |

### CFO Dashboard Events

| Event | Trigger | Key Properties |
|---|---|---|
| `cfo_dashboard_viewed` | CFO opens dashboard | date_range_selected |
| `pipeline_filter_applied` | CFO filters by stage | filter_stage |
| `revenue_at_risk_alert_viewed` | CFO views at-risk contracts | num_contracts_at_risk, total_value_at_risk |

---

## How to Read the Data

### If Activation Time Is Rising
1. Check `extraction_completed.extraction_time_ms` — is AI slower?
2. Check `review_form_opened` → `approval_submitted` delta — are ops slower on the form?
3. Check `num_fields_red` average — more red fields = more time on form
4. Check `finance_review_triggered` rate — more finance reviews = longer P95

### If Billing Error Rate Increases
1. Pull `field_edited` events — which fields are being edited after a high confidence score?
2. Those fields have a calibration problem — confidence scoring is overconfident
3. Retrain confidence model for those specific fields

### If Form Completion Rate Drops
1. Check `review_abandoned` events — when in the flow are ops leaving?
2. Check `num_red_fields` for abandoned sessions vs. completed sessions
3. If correlation: reduce friction for red field confirmation UX

### If AI Extraction Accuracy Drops
1. Check `extraction_completed.avg_confidence_score` trend
2. Check document type distribution — more scanned docs = lower accuracy
3. Check if specific field names are being edited at high rates

---

## Review Cadence

| Review | Frequency | Owner | What to Look At |
|---|---|---|---|
| Activation time + funnel | Weekly | PM | P50, P95, completion rate |
| Extraction accuracy audit | Weekly | PM + Ops | Field edit rates, most-corrected fields |
| Billing error audit | Weekly | Finance | Any post-activation corrections |
| CFO dashboard | Monthly | CFO + PM | Revenue impact, pipeline, ops efficiency |
| Full metrics review | Monthly | PM + Stakeholders | All metrics, trends, v2 prioritisation |

---

*Metrics Author: Abinaya J | Version 1.0 | February 2026*
