<div align="center">

# AI Tax System

### AI-powered tax return processing for CPA firms. OCR, Claude extraction, confidence-gated review, ProConnect and DrakeTax export.

![License](https://img.shields.io/badge/license-MIT-3b82f6)
![Built by Hyperion AI](https://img.shields.io/badge/built_by-Hyperion_AI-0f172a)
![Stack](https://img.shields.io/badge/stack-Next.js_·_FastAPI_·_Claude-64748b)

[View Design Doc (PDF)](DESIGN_DOC.pdf) · [Architecture](docs/architecture.md)

---

![Architecture](docs/architecture.svg)

</div>

## What this is

A production-safe pattern for extracting structured data from uploaded tax documents (1099s, 1040s, K-1s, Schedule C) and handing off to Intuit ProConnect or DrakeTax.

Most AI tax workflows ship as impressive demos and fall apart on the first scanned 1099 or handwritten return. This reference shows the pattern that survives real documents: OCR before LLM, per-field confidence scoring, human review queue for anything below threshold, and PII redaction before any prompt touches Claude.

## Why this exists

- **OCR before LLM** - pre-parse document structure rather than dumping pixels into an LLM
- **Confidence scoring per field** - every extracted value carries a score; nothing auto-exports below threshold
- **Human review queue** - low-confidence fields route to a CPA before anything reaches ProConnect
- **PII redaction before prompts** - SSNs and ITINs tokenised before hitting the LLM
- **Append-only audit trail** - every mutation logged immutably for compliance

## Documentation

**[Design Doc (PDF)](DESIGN_DOC.pdf)** — Branded system design with diagrams, tool choices, phases, and risks.

**[Architecture](docs/architecture.md)** — Layered view with component responsibilities and design decisions.


**[Flow Diagram](docs/flow.svg)** — Sequence diagram of the happy path.

## Stack

| Layer | Technology |
|-------|-----------|
| Web UI | Next.js 15 (App Router) |
| API Gateway | FastAPI + Pydantic |
| Workers | Celery + Redis |
| Database | Postgres |
| Object Storage | AWS S3 (encrypted, KMS) |
| OCR | AWS Textract |
| LLM | Anthropic Claude 3.5 Sonnet (zero-retention) |
| Target Systems | Intuit ProConnect / DrakeTax |

## Quick Start

```bash
pnpm install
docker compose -f docker-compose.local.yml up -d
cd apps/api && poetry install && poetry run alembic upgrade head && cd ../..
pnpm dev
```

## Who this is for

**CPA firm owners** evaluating AI tax workflow vendors - use this to understand what a production-ready pipeline looks like before committing.

**Developers hired to build one** - fork this repo as a starting point. Everything a production tax workflow needs is here in skeleton form.

**Technical reviewers** - skim the architecture and flow diagrams to understand the pattern in 5 minutes.

---

<table>
<tr>
<td width="180" valign="top">
<img src="https://hyperionai.dev/founder.png" width="160" alt="Vitalijus Alsauskas" />
</td>
<td valign="top">

### Vitalijus Alsauskas

Software and AI Engineer

Software dev from Lithuania, worked at IBM, living in Vietnam. Building enterprise-level tools for startups and companies.

[LinkedIn](https://www.linkedin.com/in/vitalijus-hyperion/) · [GitHub](https://github.com/Vitals9367) · [hyperionai.dev](https://hyperionai.dev)

</td>
</tr>
</table>

---

### Book a call

Free 30-minute scoping session for teams considering this pattern.

**[cal.com/vitalijus-alsauskas/project-request →](https://cal.com/vitalijus-alsauskas/project-request?overlayCalendar=true)**

Or reach out via [LinkedIn](https://www.linkedin.com/in/vitalijus-hyperion/).

---

## License

MIT. Fork, adapt, ship.
