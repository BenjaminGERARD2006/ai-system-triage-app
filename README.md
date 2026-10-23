# Support Ticket Triage App — Course 5

A small full-stack Node.js/Express app demonstrating chat with conversation context, structured triage, LLM/Decision Model/combined strategies, smoke testing, run logging and an admin page.

## Important scope note
The supplied ZIP contained only two course PDFs, not the existing Triage V1/V2 source code, the Teams post **C4 - Homework**, or the official labelled 100-ticket and 500-ticket datasets. The included datasets are **synthetic demonstration fixtures**, not the official course datasets. The project is therefore a working baseline implementation, not a verified replacement for the complete course project until the missing materials are provided and requirements reconciled.

## Requirements
- Node.js 18+ (built-in `fetch` required)
- A chat-completions-compatible LLM API key and model for LLM or DM+LLM strategies
- A free hosted Decision 3.0 API key for DM or DM+LLM strategies

## Setup
```bash
npm install
cp .env.example .env
# Edit .env and set LLM_API_KEY, LLM_MODEL, and DECISION_API_KEY as needed
npm start
```
Open http://localhost:3000. The Decision 3.0 key can be created at https://decision.vllm-sr.ai/. Do not commit `.env` or expose API keys in screenshots/source control.

LLM settings:
- `LLM_BASE_URL`: base URL for a provider implementing `/chat/completions` (default `https://api.openai.com/v1`)
- `LLM_API_KEY`: secret key
- `LLM_MODEL`: provider model ID

Decision Model settings:
- `DECISION_BASE_URL`: default `https://decision.vllm-sr.ai`
- `DECISION_API_KEY`: secret key
- `DECISION_MODEL`: default `vllm-sr/d3-flash`

## Features
- Triage V1 baseline and V2 context-aware prompt variants
- LLM-only, Decision Model-only, and DM+LLM strategy selector
- Server-side Zod validation for incoming triage requests and returned structured triage results
- Conversation context passed to subsequent triage requests in the browser session
- Input/output token display when supplied by the LLM provider; Decision 3.0 reports input tokens and zero generated output tokens
- Smoke-test runner supports random samples from the supplied 100-row labelled fixture or 500-row unlabelled fixture
- Accuracy and mismatch counts for labelled data; per-ticket predictions and failures
- JSON-file run logs exposed on `/admin`, including model/strategy, outcomes, latency and estimated LLM cost
- DM probabilities and security-risk signal included in the rationale; combined mode returns both model results and agreement

## Limitations / follow-up before final submission
1. Replace synthetic fixtures in `data/labelled-100.json` and `data/tickets-500.json` with official datasets from the instructor. The 500-ticket fixture is unlabelled as a smoke-test set; the labelled fixture is synthetic and must not be presented as official ground truth.
2. Add the requirements from Teams **C4 - Homework**; that post was not present in the provided ZIP.
3. Cost estimate uses a deliberately generic placeholder formula (`$1/M input tokens, $3/M output tokens`). Replace it with the selected LLM provider/model's actual prices before using the cost field as a real bill estimate. Decision-model cost is not estimated.
4. JSON-file logging is intended for a local classroom demonstration, not concurrent multi-instance production deployment. Do not send confidential support tickets to external APIs.
5. The DM+LLM strategy calls the DM first, then the LLM; the LLM result is primary and the DM result is shown for comparison.

## API routes
- `GET /api/health` — app and provider configuration status
- `GET /api/config-status` — provider setup status
- `GET /api/datasets` — dataset counts
- `POST /api/triage` — validated triage call
- `POST /api/smoke-test` — sequential random-sample smoke test
- `GET /api/admin/runs` — run history

## Tests
Run `npm test`. The tests cover schemas and deterministic utility behaviour; provider calls need valid credentials and are not invoked by the unit tests.


## C4 homework requirement checklist

- F1: `/api/triage` validates request payloads with Zod; model results are validated against the triage result schema and invalid requests return a 400 JSON error.
- F2: The triage UI displays input, output, and total tokens when provided by the provider. Decision-model APIs may not report token usage.
- F3: The smoke-test UI selects the dataset and random sample size; labelled rows display predicted and expected category/priority and summary mismatch counts.
- F4: Smoke-test runs are persisted in `data/runs.json`; `/admin` shows model, outcomes, accuracy, latency, and estimated cost.
- F5 (optional): Not required for this assignment.

**Dataset warning:** Included datasets are synthetic demo fixtures, not instructor-provided official data. Replace them with the official datasets before reporting assignment accuracy. Cost values are estimates and may not reflect provider pricing or Decision Model usage.
