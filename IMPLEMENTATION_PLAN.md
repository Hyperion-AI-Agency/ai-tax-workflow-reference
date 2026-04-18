# Implementation Plan

Phased build plan for an AI tax workflow based on this reference architecture.

---

## Problem Statement

Build a system where CPAs upload tax documents, an LLM extracts the fields they need, and the output lands in a format ProConnect or DrakeTax can import. The hard part is not the LLM call itself. It is keeping accuracy consistent across document variants (scanned 1099s, prior-year returns with handwritten notes, K-1s) and making sure PII never leaks.

This plan ships in three two-week phases, with a working skeleton at the end of week 2 and a second target system mapped by week 6.

---

## Phase 1: Foundation (week 1-2)

**Shipping:** End-to-end walking skeleton for one document type (e.g. 1099-MISC).

### Milestones

- [ ] Upload portal with signed URLs and encrypted S3 buckets (KMS-managed)
- [ ] Textract OCR wired up, raw output stored alongside original PDF
- [ ] Claude extraction for ~10 core fields with confidence scores
- [ ] Basic review queue UI (functional, not polished)
- [ ] Audit logging on every mutation (writes to immutable S3)
- [ ] Staging deployment pipeline (push to `main` deploys automatically)
- [ ] One happy-path end-to-end integration test

### Definition of done

A CPA can upload a test 1099-MISC, see extraction results in under 60 seconds, review one low-confidence field, and see the field persist.

---

## Phase 2: Coverage (week 3-4)

**Shipping:** Full field coverage for the 5 most common tax forms plus ProConnect mapping.

### Milestones

- [ ] Extraction expanded to: 1040, 1099s (MISC/NEC/INT/DIV), K-1, W-2, Schedule C
- [ ] Per-field confidence thresholds tuned against a calibration set of 10 real returns
- [ ] Mapping layer for ProConnect (module `proconnect.py` with full field mapping)
- [ ] Review UI polish: original document region displayed next to extracted value for fast verification
- [ ] Load testing at 3x expected season volume
- [ ] Error tracking wired into Sentry with Slack alerts

### Definition of done

Firm can process 50 returns through the system, with 85%+ auto-approved (above confidence threshold) and the rest reviewed in under 2 minutes per return.

---

## Phase 3: Production and Handoff (week 5-6)

**Shipping:** Second target (DrakeTax), observability, runbook, and knowledge transfer.

### Milestones

- [ ] Mapping layer for DrakeTax (module `draketax.py`)
- [ ] Sentry alerts wired into firm Slack or email
- [ ] Ops runbook: what to do when review queue fills up, how to re-process a failed return, how to rotate Claude API keys
- [ ] Backup + restore procedure documented and tested
- [ ] Knowledge transfer session with recorded walkthrough
- [ ] Production deployment with firm-owned AWS account

### Definition of done

Firm runs production autonomously. Vitalijus is on retainer for bug fixes and accuracy improvements, not day-to-day operations.

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Claude hallucinates field values on messy scans | Confidence scoring + review queue. Nothing exports without either high confidence OR human approval. Auto-approved fields still log source coordinates for retroactive audit. |
| PII leaks through LLM prompts | Anthropic zero-retention flag. SSNs and ITINs pre-redacted with reversible tokens before prompts. Audit log of every prompt. Data never leaves firm AWS except the LLM call, and that call is scrubbed. |
| Volume spike during tax season breaks pipeline | Celery workers autoscale on queue depth. Alerts at queue depth 100. Fallback to Claude Haiku under extreme load so nothing stalls. Load tested at 3x expected volume. |
| ProConnect or DrakeTax changes import format mid-season | Mapping layer isolated per target. Nightly integration tests against sample imports alert on regressions. |
| CPA correction corrupts historical data | Append-only correction log. Every correction carries user ID and timestamp. Original extraction preserved. Bad corrections are replayable from audit log. |

---

## What I Need From You

To move into Phase 1, I need:

- **5-10 real returns (redacted is fine)** to calibrate extraction accuracy on your actual document mix
- **Access to a test ProConnect or DrakeTax account** for the mapping layer
- **Field priorities** - which 20-30 fields are must-haves vs nice-to-haves (I can send a suggested list)
- **Data residency confirmation** - US-only, or flexible?
- **A 30-minute kickoff call** to walk through the plan and tighten scope before any code ships
