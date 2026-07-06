# PlainTerms

**Understand any contract before you sign it.**

PlainTerms is an AI-powered contract analyzer that flags risky and unusual clauses, explains them in plain language, and gives you a checklist of what to negotiate. It exists because legal review is priced for corporations, not for the freelancer signing a client agreement or the tenant signing a lease.

> PlainTerms explains contracts. It does not provide legal advice. See [Disclaimer](#disclaimer).

## Why this exists

Every day, people sign contracts they never fully read, because getting a lawyer to review them costs more than the contract is worth.

- A freelancer signs a client contract with a buried clause transferring ownership of all their work, including personal projects.
- A tenant signs a lease that quietly makes them liable for structural repairs.
- A designer accepts payment terms that only pay out when the client's own client pays.

PlainTerms reads the contract for you: every clause categorized, risk-scored against what is standard, explained without jargon, with concrete suggestions on what to push back on.

## Features

- Upload a PDF or DOCX. Version 1 supports freelance/service agreements and tenancy/lease agreements.
- Clause-by-clause analysis, with each clause categorized and rated as high risk, caution, or standard.
- Plain-language explanations of what each clause actually means.
- A "what's normal" comparison. Each clause is checked against a curated library of standard clause language, so flags are grounded in reference material rather than model guesswork.
- A negotiation checklist with suggested pushback for flagged clauses.
- Streaming results. Clauses appear in real time as they are analyzed.
- A split-pane viewer: your original document with inline highlights on one side, synced to detailed clause cards on the other.
- Published accuracy numbers, measured against a hand-annotated set of real contracts. See [Evaluation](#evaluation).

## Screenshots

Coming soon. The project is in active development; see the [roadmap](#roadmap).

## Architecture

```
┌──────────────────────────────────────────────────────┐
│   Next.js 14 · TypeScript · Tailwind · shadcn/ui     │
│   (Vercel)                                            │
│   Upload → streaming analysis view (SSE)              │
└──────────────────▲───────────────────────────────────┘
                   │ JSON / Server-Sent Events
┌──────────────────┴───────────────────────────────────┐
│   FastAPI · Pydantic v2   (Railway)                   │
│                                                        │
│   Analysis pipeline:                                   │
│   1. Parse      PDF/DOCX to text with char offsets     │
│   2. Segment    clause splitting (rules + LLM)          │
│   3. Classify   contract type and clause categories     │
│   4. Analyze    per-clause LLM risk assessment          │
│   5. Compare    embeddings vs standard-clause library   │
│   6. Summarize  document summary + checklist            │
└──────┬───────────────┬────────────────┬───────────────┘
       │               │                │
  Claude API      Postgres          pgvector
  (analysis)      (analyses)    (reference clauses)
```

The key design decision: every clause is compared against a curated reference library of standard clause language using embedding similarity. The model is shown what normal looks like instead of being asked to guess.

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 14 (App Router), TypeScript, Tailwind CSS, shadcn/ui, TanStack Query, Zod |
| Backend | FastAPI, Pydantic v2, SQLAlchemy |
| AI | Claude API (structured outputs, temperature 0), embedding-based clause comparison |
| Database | PostgreSQL with pgvector |
| Parsing | PyMuPDF, python-docx (offset-preserving extraction) |
| Evals | Custom harness plus a hand-annotated golden dataset |
| Deployment | Vercel (web), Railway (API and database) |

## Getting started

### Prerequisites

- Python 3.12+
- Node.js 20+
- PostgreSQL 16+ with the pgvector extension
- An Anthropic API key

### Backend

```bash
cd apps/api
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env        # add ANTHROPIC_API_KEY and DATABASE_URL
alembic upgrade head        # run migrations
uvicorn app.main:app --reload
```

### Frontend

```bash
cd apps/web
npm install
cp .env.example .env.local  # set NEXT_PUBLIC_API_URL
npm run dev
```

Open http://localhost:3000 and upload a contract, or try the built-in sample lease.

## Project structure

```
plainterms/
├── apps/
│   ├── web/                  # Next.js frontend
│   │   ├── app/              # routes (upload, analysis/[id])
│   │   ├── components/       # document pane, clause cards, checklist
│   │   └── lib/              # API client, SSE, Zod schemas
│   └── api/                  # FastAPI backend
│       ├── app/
│       │   ├── pipeline/     # parse, segment, classify, analyze, compare, summarize
│       │   ├── models/       # Pydantic schemas (source of truth)
│       │   ├── llm/          # prompts, Claude client
│       │   └── routes/
│       ├── reference_library/  # curated standard clauses
│       └── evals/            # golden dataset and eval harness
└── docs/
    ├── blueprint.md          # full build plan
    └── risk-taxonomy.md      # clause categories and risk definitions
```

## Evaluation

Most AI contract tools are prompts with a UI. PlainTerms is measured.

Every release is scored against a hand-annotated golden dataset of real freelance contracts and leases:

| Metric | Target | Current |
|---|---|---|
| Clause segmentation (boundary F1) | 0.90+ | in progress |
| Clause category accuracy | 0.85+ | in progress |
| High-risk recall | 0.85+ | in progress |
| High-risk precision | 0.70+ | in progress |

```bash
cd apps/api
python evals/run_evals.py   # regenerates the scoreboard
```

Prompt and pipeline changes only ship if eval scores hold. Scoreboard history lives in [`evals/`](apps/api/evals/).

## Roadmap

**v1 (current)**
- [x] Project blueprint and risk taxonomy
- [ ] PDF/DOCX parsing with offset preservation
- [ ] Clause segmentation (rules with LLM fallback)
- [ ] Per-clause risk analysis grounded in the reference library
- [ ] Golden dataset and eval harness
- [ ] Streaming analysis UI (SSE)
- [ ] Split-pane document viewer with synced highlights
- [ ] PDF export
- [ ] Public demo with sample contracts

**v2**
- [ ] Scanned and photographed contracts (OCR)
- [ ] Employment contracts and NDAs
- [ ] Multi-language output, including Nigerian Pidgin
- [ ] User accounts and saved analyses

**v3**
- [ ] Clause rewriting suggestions (redlining)

## Contributing

Contributions are welcome, especially:

- Sample contracts (redacted or anonymized) to grow the golden dataset
- Review of the risk taxonomy from anyone with legal or contract-heavy experience
- Frontend polish, accessibility, and test coverage

Open an [issue](../../issues) or submit a pull request.

## Disclaimer

PlainTerms is an educational tool. It explains contract language and compares it against commonly used clause patterns. It is not a law firm, does not provide legal advice, and is not a substitute for a qualified lawyer. For decisions with significant legal or financial consequences, consult a licensed attorney in your jurisdiction.

## License

MIT. See [LICENSE](LICENSE).

## Author

**Gift Ojeabulu**

Portfolio: [giftojeabulu.com](https://giftojeabulu.com)

If this project is useful to you, consider starring the repo. It helps others find it.
