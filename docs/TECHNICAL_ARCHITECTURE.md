# AI Travel Optimizer — Technical Architecture

**Status:** Proposed architecture; design only. No application implementation is part of this document.  
**Repository inspected:** Next.js App Router, React 19, TypeScript, Tailwind CSS 4, PostgreSQL, Drizzle ORM, Zod, and Vitest are already configured. Existing foundations include a trip request Zod schema, a transport provider interface, an offer validation status type, a baggage-aware comparable-price helper, an initial Trip/TransportOffer schema, and a health route. See [README](../README.md), [`src/lib/schemas/trip-request.ts`](../src/lib/schemas/trip-request.ts), [`src/server/providers/transport/types.ts`](../src/server/providers/transport/types.ts), and [`src/server/db/schema/index.ts`](../src/server/db/schema/index.ts).

## 1. Architectural decision

Use a **single Next.js application with server-only domain modules and PostgreSQL** for the MVP. Next.js route handlers provide the API boundary; Drizzle owns persistence; Zod validates all external and user input; provider adapters implement narrow interfaces; and the optimizer remains a deterministic TypeScript domain module.

This matches the repository and avoids premature services, queues, or a second database. Provider calls and route enumeration are bounded synchronous work for an initial MVP. Add a durable job system only when measured search duration, provider quotas, or retry requirements make request/response execution unsuitable. Keep provider clients swappable so that moving long-running orchestration out of Next later does not require rewriting domain contracts.

### Runtime boundaries

- **Browser:** trip form/chat UI, itinerary/map/dashboard rendering, and explicit user actions. It never receives provider secrets or raw credentials.
- **Next.js route handlers (`src/app/api/**`):** authentication/ownership checks when accounts exist, request validation, rate limits, orchestration invocation, DTO serialization, and safe error responses. No provider-specific optimization logic in route handlers.
- **Application services (`src/server/services/**`, proposed):** coordinate intake, search runs, provider selection, persistence, optimization, follow-up accommodation/activity search, and handoff preparation.
- **Domain (`src/server/domain/**`, proposed):** normalized trip requirements, transport graph, baggage-inclusive cost rules, route feasibility, candidate generation, deterministic ranking, and explicit completeness/confidence labels. Must not depend on Next.js or provider SDKs.
- **Provider adapters (`src/server/providers/**`):** server-only API clients, provider request/response mapping, documented-permission checks, timeouts, rate handling, and provider-specific validation. Adapters return canonical domain types, never provider SDK types.
- **Persistence (`src/server/db/**`):** Drizzle schema, repositories, migrations and transactions. Do not expose ORM rows as public API contracts.
- **Shared types/schemas (`src/lib/schemas/**`, `src/types/**`):** request/response schemas and safe client contracts. Do not put secret-bearing or provider SDK code in shared/browser-importable modules.

### Proposed folder layout

```text
src/
  app/
    api/
      trips/route.ts
      trips/[tripId]/route.ts
      trips/[tripId]/search/route.ts
      trips/[tripId]/runs/[runId]/route.ts
      trips/[tripId]/offers/[offerId]/validate/route.ts
      trips/[tripId]/handoffs/route.ts
      health/route.ts
    (planner)/...                 # server-rendered dashboard and planner pages
  components/                      # presentational client/server UI
  lib/
    schemas/                        # Zod request and response DTO schemas
    api/                             # typed browser fetch helpers
  types/                             # browser-safe shared primitives only
  server/
    auth/                            # user identity and ownership policy
    config/                          # validated server environment
    db/schema/                       # Drizzle entities and relations
    db/repositories/                 # persistence boundaries
    domain/
      trip/                          # normalized requirements and invariants
      money/                         # exact amount/currency operations
      transport/                     # legs, offers, baggage, trust policy
      routing/                       # candidate generation and feasibility
      ranking/                       # objectives and alternatives
    services/                        # orchestration and DTO composition
    conversation/                    # extraction interface and confirmation policy
    providers/
      registry.ts
      trust-policy.ts
      transport/types.ts
      flights/<provider>/             # flight adapter(s)
      ground/<provider>/              # rail/bus adapter(s)
      accommodation/<provider>/
      activities/<provider>/
    observability/                    # structured logging, redaction, metrics
    security/                         # rate limiting and safe error mapping
```

The paths describe target organization, not a requirement to move current files immediately. Keep current `src/server/providers/transport` and `src/server/optimization` contracts as starting points; migrate incrementally as modules are implemented.

## 2. Request and planning flow

1. **Intake:** accept either a structured trip form or natural-language message. Store the original request separately from parsed fields. A parser/LLM may propose structured values, but may not silently invent missing dates, airports, traveler counts, bag needs, currency preferences, or acceptable sellers.
2. **Normalization:** validate with Zod, resolve airport/city/place references through an authorized geocoding/catalog provider when available, normalize dates as local calendar dates plus an IANA timezone, and persist a versioned `TripRequirements` snapshot. Present ambiguous or inferred high-impact constraints for user confirmation.
3. **Search run creation:** create a `SearchRun` with a snapshot of requirements, provider registry versions, selected provider coverage, and run status. This makes later result explanations reproducible.
4. **Provider search:** choose only enabled providers with permitted-use status and matching mode/market coverage. Apply timeouts, rate limits, retry rules documented by that provider, and bounded parallelism. Persist raw response references only if terms allow it; otherwise retain normalized fields and provider IDs needed for audit/revalidation.
5. **Canonicalization:** Zod-validate provider responses at the adapter boundary, map each itinerary to `TransportLeg` and `TransportOffer`, and store fare and baggage components without erasing source currency or unknown values.
6. **Price validation:** for candidate offers, call the provider’s documented price/revalidation operation. Append a `PriceValidation` event rather than overwriting what was originally returned. A validation can report changed price, currency, conditions, expiry, availability, or missing data.
7. **Candidate generation:** build a time-constrained directed graph from permitted offer legs and supported connections. Generate a bounded set of complete routes using the user’s trip dates, required start/end, airports, allowed modes, minimum connection rules, and visit constraints.
8. **Feasibility and ranking:** reject invalid/disconnected/date-infeasible routes. Compute comparable cost only when fare, required bag charges, currencies, and required segments are known. Rank routes with explicit objective functions and confidence/coverage labels, then retain diverse alternatives (e.g. cheapest complete route, balanced route, more-places route) only where each is genuinely feasible.
9. **Persist and present:** save route candidates/scores linked to the run and return dashboard DTOs with source, retrieval and validation timestamps, uncertainty, and searched-coverage limitations.
10. **Accommodation and activities:** after initial transport selection, search around each stop’s dates/occupancy. Add estimates only as separate offers. Recompute full-trip economics while preserving the initial transport-only score and the accommodation/activity source/status.
11. **Handoff:** immediately revalidate the selected transport or referral offer, show updated price, seller, conditions, baggage and expiration, then ask for explicit approval before navigating to a supplier or invoking any order-capable API. Planning endpoints must never create an order or capture payment.

## 3. Module responsibilities

| Module | Responsibility and key boundary |
|---|---|
| Conversation and trip intake | Convert user text into candidate fields; preserve original text and extraction provenance. Missing/ambiguous constraints are questions, not defaults hidden from the user. LLM output is untrusted input and must pass schemas. |
| Trip requirement normalization | Canonical airport/place IDs, local dates and timezone, passenger/traveler set, baggage, allowed modes/providers, date flexibility, endpoint rules and optimization preference. Version snapshots so route results are tied to exact constraints. |
| Provider registry and trust policy | Registry metadata includes provider ID, modes, markets, enabled state, access/terms review date, permitted operations, seller identity policy and capabilities. Allow only authorized/documented integrations and user-approved sellers. Deny unknown or expired policy by default. |
| Flight search and repricing | Adapter accepts supported passenger/slice/date-flexibility fields, discovers offers and calls documented reprice/validation. It reports unsupported constraints rather than dropping them. Never scrape consumer sites. |
| Baggage and total-price normalization | Normalize personal item/cabin/checked allowances per traveler and leg, included quantities, dimensions/weight where supplied, paid add-ons and mandatory fees. Preserve `unknown` distinctly from zero. Only exact same-currency totals may be directly compared; FX must be explicit and timestamped. |
| Rail and bus search | Separate adapters under a shared ground-transport contract. Return operator, station, segments, scheduled local times/timezones, fares, luggage policies, seller, booking/referral behavior and validation capability. Do not assume each mode/provider returns all fields. |
| Route candidate generation | Build route alternatives from normalized stop nodes and transport legs. Bound city count, candidate count, segment count, provider calls, and search duration. Support open-jaw and multi-city endpoints without assuming departure and return airports match. |
| Route feasibility and optimization | Validate chronology, dates, stop nights, transfers, minimum connection buffers, airport/station changes, time windows and mode policy. Use deterministic constraints/objectives and retain an explanation of exclusions. |
| Alternative route ranking | Produce separate cost-first, balanced, and places-first alternatives when distinct feasible candidates exist. Avoid a single opaque composite score; expose sub-scores, weights, trade-offs, and completeness. Never say “verified cheapest” when a required fare/bag cost is missing or coverage is partial. |
| Accommodation and activity search | Run only after transport candidate selection. Query date/occupancy/room preferences and activities for stop dates. Respect each provider’s live/stale states, opening-hour unknowns, referral versus booking capability and coverage. |
| Dashboard aggregation | Compose normalized itinerary, map coordinates, local-time day plan, travel distance/duration, mode counts, layovers, spend breakdown and source/status labels. API DTOs must not leak provider tokens or unnecessary raw payloads. |
| Booking handoff | Prepare a provider-supported URL or a pending approval action. Revalidate immediately before handoff and require explicit approval. Payment and purchase are separate future scope; never purchase from a planner/search endpoint. |
| Logging, rate limiting, errors | Structured logs with correlation ID, `tripId`, `searchRunId`, provider and latency; redact PII, prompt text where unnecessary, tokens and payment data. Enforce per-user/IP and per-provider quotas, timeouts, bounded retries, circuit breaking if justified by measured failure, and stable client-safe error codes. |

## 4. Provider and trust policy

Provider implementation depends on successful account approval, documented endpoint permissions, and an explicit data contract. Keep capabilities declarative and validated: supported modes, passenger/occupancy support, multi-slice support, baggage fields, validation operation, booking/referral method, and covered markets. A capability marked unknown is not true.

At minimum, seller policy checks:

- Provider integration is authorized and currently configured for the requested use.
- The returned seller/merchant identity is known and matches the user’s allow-list or approved-provider policy.
- The booking/referral URL is provider-supplied or constructed only using documented parameters; validate scheme and allowed host before returning it.
- The offer is current enough for search display and revalidated before handoff.
- Booking-capable endpoints are inaccessible to ordinary search requests and are gated by explicit user confirmation and future payment/identity policy.

The Google Flights consumer page is not a structured-offer provider. No undocumented endpoint, scraping, browser automation, or access-control bypass belongs in the provider registry. Partner applications require explicit written approval and documented API terms before being treated as a candidate integration.

## 5. Optimization design

Model each normalized location/airport/station as a graph node and each schedulable provider segment or through-itinerary as a directed time-dependent edge. An edge includes departure and arrival instants, local timezone context, seller, mode, fare and baggage completeness. Candidate routes are paths through the graph between required start and end, optionally with an open-jaw return endpoint, subject to trip time and user constraints.

For MVP, use bounded candidate generation over the destinations and stop count the user confirmed or the system proposed. Do not attempt unbounded global routing. Feasibility constraints run before ranking. Important constraints include:

- Each selected edge is an actual provider-returned offer, with dates matching the selected leg.
- Local departure/arrival times map to unambiguous instants; DST ambiguity is preserved/resolved explicitly.
- Arrival precedes the next departure with a mode/location-specific connection buffer.
- Ground transfers between different stations/airports require a modeled transfer estimate or are marked infeasible/unknown.
- All travelers, required baggage, seller policy, allowed modes and date windows are satisfied.
- Route contains the requested endpoints and fits the whole-trip date window.

Rank separate objective profiles rather than treating unknown prices as very high, zero, or imputed. Cost score uses exact comparable transport totals. Balanced score combines disclosed normalized components such as transport spend, total transit time, number of places, overnight/transit burden, and source completeness. More-places score maximizes unique places subject to feasibility and user stop preferences. Persist the objective version and score components. Tie-break deterministically for reproducibility.

Labels must describe evidence:

- **Comparable total:** all mandatory known costs in a single currency or with a dated FX conversion policy.
- **Incomplete price:** any fare, required bag charge, currency conversion, or segment cost is unknown.
- **Cheapest found in searched coverage:** best complete candidate among the recorded providers/markets/dates only.
- **Supplier revalidated:** provider’s documented validation succeeded at the timestamp shown.
- **Booking readiness confirmed:** use only if supplier explicitly returned a readiness/booking token or equivalent; this still does not mean a booking exists.
- **Stale/unavailable:** provider validation or expiry indicates no longer usable.

Accommodation and activity prices are separate cost categories, with their own timestamps, occupancy/date conditions, cancellation conditions, and source. Re-evaluate full-trip economics after adding them; do not overwrite the transport-only comparison.

## 6. API and persistence strategy

Use versioned, resource-oriented route handlers. Proposed first endpoints:

- `POST /api/trips` — create/update a trip from validated structured requirements and optional natural language.
- `GET /api/trips/:tripId` — return authorized trip requirements and latest dashboard summary.
- `POST /api/trips/:tripId/search` — create a SearchRun and start bounded search/optimization for supported providers.
- `GET /api/trips/:tripId/runs/:runId` — retrieve progress/result/errors and provider coverage.
- `POST /api/trips/:tripId/offers/:offerId/validate` — request documented repricing and append validation record.
- `POST /api/trips/:tripId/handoffs` — prepare handoff only after current validation and an explicit approval signal; purchase endpoint is deliberately out of scope.

All request JSON and provider responses are parsed with Zod. Return stable error shapes (`code`, safe `message`, correlation ID, retryability) without provider stack traces. Use idempotency keys for trip creation/search-run submission to prevent duplicate searches on client retries. Authenticate and enforce trip ownership before exposing saved trips; anonymous MVP can use a signed opaque session only if its security properties are reviewed.

Persist normalized requirements, search runs, canonical offers, validations, candidates and scores. Use PostgreSQL transactions for coherent run-state transitions. Index ownership/time queries, provider offer lookup IDs, run status and route candidate ranking. Keep immutable `PriceValidation` observations and requirement snapshots instead of mutating history. Store provider raw payload only when contracts permit, encrypted/retained minimally; normalized source IDs and hashes may be enough for audit.

## 7. Security and operations

- Validate environment variables at server startup; keep credentials in server-only modules and deployment secret storage. Never use `NEXT_PUBLIC_` for provider or database credentials.
- Apply request-size limits, Zod validation, auth/ownership checks, origin/CSRF policy for state-changing routes, and per-user/IP request throttles. Start with a PostgreSQL-backed or in-process throttle appropriate to a single instance; use a shared limiter only when deployment scale requires it.
- Respect supplier quotas and terms. Use provider-specific concurrency, timeout, retry-after, and retry policies; never retry purchase/order endpoints automatically.
- Use structured errors and logs with correlation/run/provider IDs. Redact names, emails, passport/contact data, raw prompts where not needed, tokens, and payment details. No payment data should be stored in this planning phase.
- Record provider terms/access review and effective date. Disable adapters when credentials or allowed purpose are missing; return an honest unavailable-provider reason instead of fixture/live-looking output.
- Development fixtures must be labeled simulated at the API and UI boundary and cannot share the live-provider status path.

## 8. Testing strategy

Vitest is already present. Add tests in dependency order:

- Unit: Zod validation, normalization, exact money/currency arithmetic, baggage completeness, seller policy, timezones/DST, feasibility, and deterministic ranking.
- Adapter contract tests: provider response fixtures (synthetic or documented sanitized examples), malformed payload rejection, missing baggage/fee preservation, expiry/status mapping, timeouts, rate limit handling, and no-secret serialization.
- Integration: PostgreSQL repositories/migrations, ownership boundaries, search-run state transitions, offer validation history, and route handler DTO/error behavior. Use a disposable test database only when configured; no real supplier purchase calls.
- End-to-end later: structured planner input through simulated provider adapters, asserting visibly simulated labeling and ranking semantics. Production smoke checks may make read-only supplier searches only under approved credentials and explicit safe quotas.

## 9. Explicit non-goals for this architecture phase

No provider code, LLM vendor, authentication product, map/geocoding vendor, payment processor, booking transaction, queue, microservice, or new datastore is selected here. Provider research and actual access agreements determine which adapters can be built. No fixtures may be represented as live travel offers.

## 10. Implementation plan overview

See [DATA_MODEL.md](./DATA_MODEL.md) for entity definitions and invariants. Implement in the order in that document’s dependency plan: canonical money/time/types and schemas; relational persistence; trust registry; provider contract/fixtures; one provider at a time; route feasibility before ranking; API and dashboard composition; lodging/activity enrichment; then handoff gates. The first implementation milestone should establish a truthful, persisted, testable search-run lifecycle, not a polished global itinerary promise.
