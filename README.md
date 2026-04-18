# AI Tax Workflow - Reference Architecture

Reference architecture for AI-powered tax return processing. Demonstrates a production-safe pattern for extracting structured data from uploaded tax documents (1099s, 1040s, K-1s, Schedule C) and handing off to Intuit ProConnect or DrakeTax.

Built by [Vitalijus Alsauskas / HyperionAI](https://www.linkedin.com/in/vitalijus-hyperion/) as a public reference for CPA firms considering AI document pipelines.

---

## Why this exists

Most AI document workflows ship as impressive demos and then fall apart on the first scanned 1099 or handwritten return. This reference shows the pattern that survives real documents:

1. **OCR before LLM** - pre-parse the document structure instead of dumping raw pixels into an LLM
2. **Confidence scoring per field** - every extracted value carries a score; nothing auto-exports below the threshold
3. **Human review queue** - low-confidence fields route to a CPA for verification before anything reaches ProConnect
4. **PII redaction before prompts** - SSNs and ITINs are tokenised before ever hitting the LLM
5. **Append-only audit trail** - every mutation logged immutably for compliance

---

## Documentation

| Document | Purpose |
|----------|---------|
| [docs/architecture.md](docs/architecture.md) | Layered architecture + component responsibilities |
| [docs/architecture.svg](docs/architecture.svg) | Architecture diagram (rendered) |
| [docs/flow.svg](docs/flow.svg) | Sequence diagram showing end-to-end return processing |
| [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) | Phase-by-phase build plan with milestones and risks |
| [CLAUDE.md](CLAUDE.md) | Operational rules for Claude Code working in this codebase |

---

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

---

## Quick Start (local dev)

```bash
pnpm install
docker compose -f docker-compose.local.yml up -d
cd apps/api && poetry install && poetry run alembic upgrade head && cd ../..
pnpm dev
```

---

## Who this is for

**CPA firm owners** evaluating AI-powered tax workflow vendors - use this as a reference to understand what a production-ready pipeline looks like before committing.

**Developers hired to build one** - fork this repo as a starting point. Everything a production tax workflow needs is here in skeleton form.

**Technical reviewers** - skim the architecture and flow diagrams to understand the pattern in 5 minutes.

---

## About

Built by Vitalijus Alsauskas. Ex-IBM (4 years, Fortune 500 clients including AskProcurement, a chatbot pulling structured Dun & Bradstreet data for procurement teams). Currently running Claude in production on Fit7D and shipping AI features on Adboard (Next.js + Supabase + Claude API).

If you are a CPA firm considering this pattern, I offer a free 30-minute scoping call - reach out via LinkedIn or [vitalijus.io](https://vitalijus.io).

- GitHub: https://github.com/Vitals9367
- LinkedIn: https://www.linkedin.com/in/vitalijus-hyperion/

---

## License

MIT. Fork, adapt, ship.
