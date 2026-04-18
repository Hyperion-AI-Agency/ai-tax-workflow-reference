# CLAUDE.md

This file is the operational foundation for Claude Code. Read it at session start. Follow it exactly.

---

## What This Project Is

Reference architecture for AI-powered tax return processing. Demonstrates the pattern of OCR + Claude structured extraction + confidence scoring + human review queue before export to ProConnect or DrakeTax.

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

## Commands

| Command | Purpose |
|---------|---------|
| `/test` | Run tests |
| `/lint` | Run linter |
| `/dev` | Start local dev environment |

---

## Conventions

- All extracted fields must carry a confidence score

- Low-confidence fields drop into the review queue, never auto-export

- PII (SSN, ITIN) redacted before any LLM call

- Every mutation logged to immutable audit bucket

- Mapping layer is isolated per target (proconnect.py, draketax.py)

- No secrets in code - use .env and GitHub secrets

---

## Operational Rules

1. **Never hardcode secrets.** API keys, passwords, connection strings go in environment variables or secret managers.
2. **Follow existing patterns.** Read existing code before modifying. Match the style.
3. **Test before committing.** Run the test suite and linters before pushing.
4. **One concern per change.** Keep PRs focused on a single feature or fix.
