# Gemini trip conversations

Set a server-only `GEMINI_API_KEY` from Google AI Studio in `.env.local` and select `INTAKE_PROVIDER=gemini`. The configured model is `GEMINI_MODEL=gemini-3.5-flash-lite`. The adapter also permits `gemini-3.1-flash-lite` and `gemini-2.5-flash-lite` when explicitly selected; it never switches models or providers after an error.

Keep the Google project on the **Free** tier without attaching billing. The app cannot determine a project's billing tier from an API key, so choosing a model with a free tier does not by itself guarantee that a billed Google project will incur no charges. This integration does not enable billing, grounding, Google Search, tools, caching storage, batch jobs, reservations or purchases.

## Request handling

Before choosing a provider, the dispatcher handles safe, complete answers to the active question locally. This includes confirmations, city counts, moving pace, explicit duration units, date flexibility, simple adult counts, a single missing child/infant age with explicit units, named bag counts and delegated city selection. These turns consume no Gemini quota. Compound requests, negations and ambiguous replies retain model interpretation. See the [efficiency benchmark](EFFICIENCY_BENCHMARK_2026-10-22.md) for real local HTTP measurements and limitations.

- Uses Google's fixed HTTPS `generateContent` endpoint. The key is a request header, never a client-side variable or URL parameter.
- Sends the bounded trip context, recent conversation and one active question. It requests one JSON response, at most 4,096 output tokens, with a 30-second timeout.
- Uses a compact Google output schema. Full Zod validation and the shared trip rules still check every proposed change atomically before saving it. Provider errors preserve the draft.
- Does not retry automatically or fall back to Groq or another service. Quota errors show Google's retry delay when supplied.
- Readiness checks remain local and consume no chatbot quota. A fresh, complete validated conversation queues a one-time flight search and opens the dashboard directly, without a redundant readiness HTTP request. Manual searches of saved drafts still perform repair/readiness review. The model itself has no supplier-search tools; incomplete trips keep the next required question.

## Usage limits

Google's per-project free-tier limits remain authoritative. Check the project's active limits in AI Studio; these vary by model/project and can be lower than the app's safeguards.

Gemini has its own durable counter in ignored `work/gemini-intake-usage.json`, separate from Groq's 6/minute and 100/day counter. The Gemini defaults are local safeguards of 30 requests per rolling minute and 1,000 per Pacific calendar day, configurable via `GEMINI_CHAT_MAX_PER_MINUTE` (1–60) and `GEMINI_CHAT_MAX_PER_DAY` (1–10,000). These numbers are **not** promised Google quotas. Failed requests count locally too. Counters are not reset when restarting the server.

Explicit `INTAKE_PROVIDER=local` selects the deterministic parser. `auto` selects a present Gemini key first, then an enabled Groq key, otherwise the local parser. Existing saved Groq conversations remain readable and can continue with Gemini.

References: [Google API](https://ai.google.dev/api/generate-content), [pricing](https://ai.google.dev/gemini-api/docs/pricing), [project rate limits](https://ai.google.dev/gemini-api/docs/rate-limits).
