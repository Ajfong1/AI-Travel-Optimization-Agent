# Conversational trip planning

Gemini is now available as a separate provider. See [Gemini setup](GEMINI_CHAT.md). Select `INTAKE_PROVIDER=groq` to use the Groq configuration described below; `auto` prefers a configured Gemini key. There is no automatic provider fallback after an error.

The planner uses Groq when `GROQ_API_KEY` is present and `GROQ_INTAKE_ENABLED` is not `false`. The default model is `openai/gpt-oss-120b`; `openai/gpt-oss-20b` is the only other allowed model. Keys stay on the server.

Each request sends the current trip values, unconfirmed fields, the last ten conversation turns, the active question, today's date and the user's original wording. Conversation history is saved with the trip in browser storage. Start a new search to reset it.

Groq returns a strict JSON envelope containing proposed changes, a short reply and at most one follow-up question. Each change's JSON-encoded value is separately validated against the application's existing field schema. The server creates provenance and confirmation metadata; the model cannot supply it. Changes are applied atomically, and the existing date, route, traveler and baggage checks determine search readiness.

The model has no search, booking, payment or browsing tools. A completed validated conversation automatically queues a flight search, shows a loading indicator and opens the dashboard Transport section. The model has no supplier-search tools. Manual **Search flights** remains available for drafts and retries. The planner contains the conversation; flight settings, progress, comparisons and results live on the dashboard. Search readiness is rechecked locally when the button is clicked, without a Groq request. Location repairs bypass Groq entirely.

## Free-tier operation

Keep the Groq account on its **Free** tier. A model name alone cannot prove the account's billing tier; an upgraded Groq account can bill the same API call. This app does not upgrade accounts or manage billing.

- Fixed Groq endpoint; no OpenAI API or other provider fallback.
- Local cap: six calls per rolling minute and 100 per UTC day, persisted in ignored `work/groq-intake-usage.json`. Failed calls count too. Groq's token quotas may stop requests sooner.
- Thirty-second timeout, bounded context and output, no automatic retries.
- Authentication, rate-limit, truncated-response and validation errors preserve the existing trip and leave the user's answer available for retry.
- With no key, the existing local parser remains available. With a configured key, provider errors are displayed rather than silently switching parsers or services.

Provider references: [Groq quickstart](https://console.groq.com/docs/quickstart), [structured outputs](https://console.groq.com/docs/structured-outputs), [free-tier limits](https://console.groq.com/docs/rate-limits), [billing](https://console.groq.com/docs/billing-faqs).
