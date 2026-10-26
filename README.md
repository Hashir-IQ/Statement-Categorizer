# Ledger Lens

Ledger Lens is a statement categorizer for a freelance designer preparing a business credit-card CSV for tax time. Upload a statement, see every transaction normalized and categorized, resolve the rows that are not safe to guess, and export a finished ledger.

The app uses fake data only. The built-in demo is synthetic and contains no account numbers, personal details, or real bank statements.

## Run the web app

```bash
pnpm install
pnpm dev:static
```

Open the local Vite URL shown in the terminal. The page starts with the synthetic demo statement so the full workflow is visible immediately. Use **Import statement** to load a CSV from your own test fixture. The supported core fields are a date, description/merchant, amount, and optional transaction type (`Debit` or `Credit`). Header aliases such as `Posted Date`, `Merchant`, `Value`, and `Debit/Credit` are accepted.

Useful checks:

```bash
pnpm check
pnpm test
pnpm build:static
python3 -m unittest discover -s backend -p 'test_*.py'
```

## Python backend

The canonical rule engine is in `backend/categorizer.py` and uses only the Python standard library. To run its small HTTP API:

```bash
python3 -m backend.server
curl -X POST http://127.0.0.1:8787/api/categorize \
  -H 'Content-Type: text/csv' \
  --data-binary @fixtures/demo_statement.csv
```

The API returns normalized transaction objects with `category`, `status`, `confidence`, `rule_id`, `rule_label`, and `review_reason`. The browser demo mirrors the same ordered rule table in `client/src/lib/categorizer.ts` so the static Preview is fully interactive without a separate service.

## How categorization works

The brain is code, not a large AI model. It checks data quality and exact duplicates first, then ordered rules for transfers, refunds, recognized income, fees, software, travel, meals, office supplies, equipment, and advertising. Anything unmatched or ambiguous becomes **Needs review**. A refund stays a refund; it is never treated as income. Every automatic result includes the exact matching rule text. See [DESIGN.md](DESIGN.md) for the full rule order and product design rationale.

## Repository guide

- `client/src/pages/Home.tsx`: interactive upload, review, summary, and export experience.
- `client/src/lib/csv.ts`: CSV parsing, flexible header mapping, validation, and export.
- `client/src/lib/categorizer.ts`: browser-side deterministic rule mirror.
- `backend/categorizer.py`: canonical Python rule engine and CSV parser.
- `backend/server.py`: standard-library HTTP API wrapper.
- `backend/test_categorizer.py` and `client/src/lib/csv.test.ts`: tests.
- `fixtures/demo_statement.csv`: synthetic fixture.
- [DESIGN.md](DESIGN.md): use case, design system, and rule decisions.
- [AI_USAGE.md](AI_USAGE.md): AI assistance, corrections, and usage-log disclosure.

No keys, passwords, tokens, or real statements are committed.
