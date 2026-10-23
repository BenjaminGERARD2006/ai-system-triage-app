# LLM–Decision Model Integration Report

## 1. Objective
Integrate at least one hosted decision model into a support-ticket triage app, let the user choose LLM-only, decision-model-only, or combined processing, run smoke tests through the selected strategy, and record which model was used. The report should be updated with actual experimental results after the official course datasets and API credentials are available.

## 2. Decision-model research

| Option | Interface | Availability / trade-off | Decision |
|---|---|---|---|
| vLLM Semantic Router Decision 3.0 | `POST https://decision.vllm-sr.ai/v1/systemone`; model IDs include `vllm-sr/d3-flash` and `vllm-sr/d3-turbo` | Hosted free API with key; typed choice/noul/score questions; quotas and rate limits apply; input may be logged | **Selected for this baseline** because the course slides describe it and current documentation exposes a direct typed-question endpoint |
| OpenAI Decisions API | `POST /v1/decisions`; documented model `gpt-6-luna` | Public beta documented; requires an OpenAI API key and may be paid, so not selected as the free option | Alternative if account access/budget is available |
| TypeSafe / Jev through OpenRouter | OpenRouter documents a System One-style decision endpoint for Jev | Provider-specific contract and billing/availability can vary | Alternative; would require a provider adapter and re-testing |

Sources checked 23 October 2026:
- vLLM Semantic Router Decision API docs: https://decision.vllm-sr.ai/docs
- Decision 3.0 hosted service: https://decision.vllm-sr.ai/
- OpenAI Decisions API reference: https://developers.openai.com/api/reference/resources/decisions/methods/create

Hosted-service quotas and pricing can change. Recheck the live docs and terms before final submission. The decision endpoint is not interchangeable with a chat-completions endpoint; it accepts a state and typed questions, returning structured answers and probabilities.

## 3. Integration strategy

### Decision-model-only
The app sends the ticket text as `state` with three typed questions:
- `category`: `choice` over billing, technical, account, returns, shipping, sales, and other
- `priority`: `choice` over low, medium, high, and critical
- `security`: `noul` estimating whether a security incident or account compromise is described

The backend maps the category and priority choices to the app's triage schema and includes probability information in the rationale. The incoming request and final triage object are validated with Zod.

### LLM-only
The backend calls an OpenAI-compatible `/chat/completions` endpoint and requests a JSON object. The response is parsed and validated against the category/priority/summary/rationale schema. Triage V1 uses a baseline prompt; Triage V2 additionally uses the recent conversation history and asks the model to acknowledge uncertainty without inventing facts.

### Combined DM + LLM
The app calls the decision model and then the LLM. The LLM result is the displayed primary result; the DM result is returned side-by-side, and an agreement flag compares category and priority. This enables later empirical comparison without pretending that the DM is automatically more accurate.

## 4. Smoke testing and logging

The frontend lets the user select dataset, random sample size, strategy, and triage version. Each sampled ticket is processed and shown with its predicted result. Labelled rows are compared with expected category and priority, with separate accuracy and mismatch counts. Failures are recorded rather than silently discarded. `/admin` lists run strategy, model name, outcome counts, accuracy, latency, and an estimated cost field where available.

**Dataset caveat:** the supplied ZIP did not contain the official C4 homework post or the official labelled 100-ticket / 500-ticket datasets. The included JSON files are synthetic demo fixtures. Results from them are not valid evidence of course-dataset accuracy. Replace them with the instructor-provided files before reporting final performance.

## 5. Evaluation plan

After replacing the demo fixtures:
1. Run the same random sample and record its ticket IDs for each strategy.
2. Compare category accuracy, priority accuracy, failure rate, median/mean latency, and model-reported token usage.
3. For combined mode, calculate DM–LLM agreement and inspect disagreements manually.
4. Compare performance on the labelled 100-ticket dataset. The unlabelled 500-ticket dataset can measure latency, failures, and qualitative outputs, but cannot produce accuracy without labels.
5. Record the model IDs, date, sample size, dataset version, API limits, and any rate-limit or timeout errors.

## 6. Current results

No genuine hosted-model experiment has been run as part of preparing this code because no API credentials were supplied. Therefore, this report intentionally does not claim accuracy, latency, or cost results. The app records these when you run it with valid credentials. The displayed cost estimate is a generic placeholder and must be replaced with the actual selected LLM provider's pricing before treating it as a financial result; Decision 3.0 cost is not estimated here.

## 7. Security and limitations

- API keys stay in server environment variables and are never sent to the browser.
- Ticket text is untrusted; the prompt instructs the LLM not to follow instructions embedded in tickets.
- The hosted Decision 3.0 documentation states that requests and responses may be logged and used to operate, analyse, improve, or train models. Do not submit sensitive or confidential customer data.
- JSON-file logs are appropriate for a local demo only; use a database and access controls for deployment.
- This is a baseline implementation. The missing C4 Homework specification and existing Triage V1/V2 source project may require additional features or changes.
