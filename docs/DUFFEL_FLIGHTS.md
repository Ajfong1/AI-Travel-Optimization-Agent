# Duffel flight search adapter

The first flight adapter targets Duffel because the provider research identified its documented multi-slice offer requests, offer retrieval, place suggestions, cursor-paginated offers, and single-offer refresh. This is a provider integration, not an assertion that this application has a Duffel production account or commercial approval.

## Required configuration and access

Set these server-only environment variables:

```dotenv
DUFFEL_ACCESS_TOKEN=...
DUFFEL_FLIGHT_SEARCH_ENABLED=true
DUFFEL_ALLOW_TEST_MODE=false
```

Obtain the API token from an authorized Duffel account and confirm the account's permitted use, production/live mode, supplier and market coverage, and commercial terms with Duffel. Do not expose the token to browser code or use a `NEXT_PUBLIC_` variable. The adapter fixes the API origin to `https://api.duffel.com`; it does not allow caller-controlled provider URLs.

Flight search is disabled by default in `.env.example`. The 2026-10-18 audit demonstrated a configured local test-mode request; that local configuration is not included in the repository and does not establish live credentials or account authorization. Duffel test-mode inventory is rejected by default; it can only be enabled explicitly in non-production using `DUFFEL_ALLOW_TEST_MODE=true` and is returned with `retrievalMode: "provider_test"`. Production rejects it.

## Server endpoints

- `GET /api/flights/places?query=London` — Duffel Places suggestions for airports/cities.
- `POST /api/flights/search` — JSON `{ "mode": "initial", "request": { ... } }` creates a request and returns the first cursor page. Passenger entries are explicit adults or age-based child/infant categories. Each slice can use an airport or city IATA code; one slice is one-way, two are round-trip, and multiple slices support open-jaw/multi-city.
- `POST /api/flights/search` — JSON `{ "mode": "page", "pageToken": "..." }` gets a subsequent cursor page. The opaque, signed token binds the cursor to the original baggage and connection settings and expires after 20 minutes.
- `POST /api/flights/offers/{offerId}/validate` — JSON `{ "validationToken": "..." }` calls Duffel's latest-offer endpoint with available services to refresh price and exposed baggage information. The signed token binds the offer and original baggage requirements and expires after 20 minutes. A successful refresh is not booking confirmation.

Search accepts cabin class and maximum connections. Date flexibility is explicitly rejected when nonzero: the documented Duffel offer request has exact slice departure dates. Month-and-duration searches are handled separately by the application's date-pair comparison described below. There is no automated order creation, payment, or handoff URL generation.

## Month and duration comparison

The planner accepts requests such as `From SFO to LAX for 3 days in December 2026, 1 adult, no bags`. It interprets three days as three calendar days / two nights, and searches outbound and return departure dates entirely within that month. When the trip endpoint is the outbound destination, this month comparison returns to the departure airport. An explicit different trip endpoint is preserved. If the year is omitted, the planner asks to confirm the inferred year before searching. A new month-and-duration request replaces previously selected exact dates.

`POST /api/flights/month-search` takes `month` (`YYYY-MM`), `nights` (0–30), and a validated flight `request` with two slices. It streams newline-delimited JSON progress events and a final result or error. A December three-day request searches 29 date pairs, with three supplier searches running at a time. Past departure dates are skipped. The process allows one active month comparison at a time; the existing API rate limit also applies to its entry point. Cancellation stops launching further searches; already-issued supplier requests finish within their existing timeouts.

The shortlist contains the five cheapest returned offers per currency with verified inclusion of the requested checked bags and carry-ons. Stale, expired, unpriced, and baggage-incomplete offers are excluded. Currencies are ranked separately. Each date search currently examines the first supplier page (up to 200 offers), so this is a comparison of returned eligible fares rather than a globally verified cheapest guarantee. Failed date pairs are counted and shown as incomplete coverage. Mode and permission failures halt the comparison; test and live offers are never mixed. Displayed search results still need current validation before any booking handoff.

Clarification answers include an explicit date field in the intake API request. A bare date answering the final-arrival question updates `returnDate` and preserves `departureDate`; confirming an inferred return year also leaves an unconfirmed departure year untouched.

## Saved results and dashboard comparison

The flight controls expose `/api/flights/connection`, which reports credential mode/configuration readiness without returning the token or sending a provider request. A live prefix is configuration evidence, not proof of account authorization or live inventory. A new supplier search is required to replace saved mock results; historical quotes keep their original mode labels.

For a no-charge requirement, set `DUFFEL_LIVE_DATA_BLOCKED=true` and leave flight search disabled. This blocks every outgoing Duffel request made with live credentials before any network call, including place lookups, searches and order paths. Merely enabling the search flag does not bypass the cost block. The current app implements no flight purchase/payment flow. Live data is not assumed free: [Duffel pricing](https://duffel.com/pricing) includes an excess-search fee of $0.005 per search beyond a 1,500:1 monthly search-to-order ratio. A no-booking requirement therefore does not itself guarantee free search access. No live verification request should be sent under a zero-charge requirement without confirmed no-charge account terms.

Multi-city comparisons now retain a five-offer shortlist **for every searched city order**, in `cityRoutes`, including entries with no eligible fares and their search coverage. The old global shortlist remains as a preview for older UI consumers, but no longer controls which city itineraries can be browsed. Progress resumes at the next unsearched city order, preserving all completed route shortlists. The planner and dashboard share a two-stage comparison: at most two recommendations (cheapest and best flights), plus a compact selector for all saved city orders; selecting an itinerary opens its four/five returned flight combinations, and choosing a fare separately adds transport to the plan. The selected comparison itinerary persists across reloads; a new batch returns the UI to city comparison so newly added routes are visible.

“Best flights” minimizes the sum of the fare/minimum-fare ratio and total-journey-time/minimum-time ratio within the same currency and inventory mode. Journey time includes layovers within each supplier slice, excludes city stays, and requires valid grouped UTC instants; fewer connections and then price break ties. Missing timing is not ranked as zero. Month and city-route shortlists retain the cheapest, balanced-best and fastest returned offers before filling remaining slots by price, so a faster but more expensive option is not discarded solely for falling outside the five lowest fares. Recommendations cover returned eligible inventory, not global optimality. If one offer wins both categories, it appears once with both labels.

Legacy searches saved only five fares globally and cannot reconstruct discarded route details locally. “Recover earlier city itineraries” re-searches only already-counted city orders missing per-route details, with progressive saves and pause support; it does not advance the next-route cursor or double-count coverage. Normal continuation then adds the next six city orders. No synthetic offers are used to fill missing historical results.

Completed exact-date and month searches are stored with their originating saved trip in browser local storage. Navigating to the dashboard or reloading restores the same supplier offers in Transport options and Route comparison. A search finishing after the user switches trips does not change the active trip. Results saved before this feature require a new search because they were previously held only in component state.

The dashboard highlights the lowest returned baggage-inclusive fare and shortest total flight time per currency and inventory mode. Flight time sums segment durations and excludes layovers and time spent at a destination; it is not a full route-feasibility assessment. Expired, stale and baggage-incomplete fares do not enter the comparison. Any change to trip requirements marks previous saved results as needing refresh while preserving them for reference. Empty completed searches replace previous results instead of retaining old fares. Test inventory remains visibly marked and never establishes a verified cheapest or bookable route.

The planner and dashboard share flight cards with larger prices, airport codes, local departure/arrival times, human-readable dates, source airport names, flight numbers and durations where supplied. Provider pagination and validation tokens are not stored in these display snapshots. Browser storage can be cleared or fill up; the planner reports saving failures while keeping results visible for the current page session.

## Offer interpretation and limitations

Each normalized segment now retains its supplier slice index (`journeyIndex`), aircraft, operating carrier and terminal details where supplied. Expand a journey on a flight card to see all its segments and layovers. Connection time uses absolute UTC arrival/departure instants, including overnight/timezone differences. Multi-day gaps between different slices are city visits, not layovers. Older saved quotes without slice grouping ask for a refresh rather than guessing connections. The persistent “Direct flights only” control sends `max_connections=0` on both offer creation and listing, including all middle legs and every city/date comparison. It means no connecting flights, not a promise about unreported technical stops.

The route map follows each actual segment's origin and destination, adds direction arrows, lists the journey order and distinguishes airport changes from flights. “Between cities” focuses on intermediate stops; “Full route including home” restores outbound and return flights. This avoids hiding European connections behind overlapping transatlantic lines.

Normalized offers retain the Duffel offer ID, retrieval timestamp, live/test mode, total/base/tax amounts and currencies, offer owner, marketing and operating carrier per segment, flight number, local departure/arrival time and airport timezone, connections/layovers, fare change/refund conditions, expiry, and exposed baggage allowances.

Duffel documents `owner` as the airline that provided an offer. It is not treated as verified booking seller identity. The actual booking seller is `null` unless a later authorized integration contract provides explicit seller evidence; no airline booking URL is synthesized. The central provider registry currently records Duffel as documentation-reviewed with no seller approval, so these results must not be represented as approved booking options.

Duffel's API exposes included baggage where returned and available baggage services on a refreshed offer. The current adapter only treats required bags as covered if per-passenger, per-segment allowance data explicitly covers every requested bag. It does not infer missing allowances or select/price optional baggage services. If data is missing or insufficient, the offer is `incomplete`, and the baggage-inclusive price is not comparable. Extra service prices are exposed by Duffel in responses but need an explicit traveler selection and fare reprice flow before they can be added to a total.

The API's single-offer GET returns current offer data but explicitly does not guarantee availability at booking. `search_result_retrieved`, `repriced_or_validated`, `stale`, and `incomplete` remain distinct statuses. Order creation is deliberately not implemented.

Provider HTTP calls have a 25-second timeout, disallow HTTP redirects, and use a fixed API hostname. Read-only GET calls use at most two retries for network failures, HTTP 429, and 5xx; offer-request POSTs retry only a definite 429 to avoid duplicate search resource creation on ambiguous failures. Quota pacing shares the provider's `ratelimit-remaining` and HTTP-date `ratelimit-reset` window across concurrent clients in the same process and credential/endpoint group. A 429 honors the full reset/Retry-After delay rather than truncating it to two seconds. See [Duffel response handling](https://duffel.com/docs/api/overview/response-handling). Search result lists use Duffel cursor pagination in pages up to 200; the app client chooses when to request a next page. Opaque HMAC-signed state is bound to the server credential, so callers cannot alter pagination or baggage requirements for validation. Errors are sanitized and response bodies/secrets are not logged. A process-local 20-request/minute API guard is included as an MVP fallback. It is not shared across server instances; deploy behind a trusted reverse proxy and add edge/shared rate limiting before horizontal production scale.

## Official documentation checked 2026-10-18

- [Places suggestions](https://duffel.com/docs/api/places/schema)
- [Offer requests, passengers, slices and cabin class](https://duffel.com/docs/api/v2/offer-requests)
- [Offer list pagination and sorting](https://duffel.com/docs/api/v2/offers/price-offer)
- [Offer schema and latest-offer retrieval](https://duffel.com/docs/api/offers/schema)
- [Adding extra bags and service relationships](https://duffel.com/docs/guides/adding-extra-bags)
- [Duffel test integration behavior](https://duffel.com/docs/api/overview/test-your-integration)

## Test-mode validation 2026-10-18

The configured Duffel test-mode credential was used successfully against the running application. The live HTTP search endpoint returned 44 supplier test offers for a one-adult SFO–LHR request, including one checked bag and one carry-on. A selected offer was then retrieved through the application's signed validation endpoint; Duffel returned `repriced_or_validated`, preserving the USD total, the satisfied baggage allowances, and the UTC/local schedule fields. No order was created and no purchase was attempted.

These responses prove authenticated test-mode search and offer refresh only. Duffel explicitly states that test-mode offers and schedules are not representative of live inventory or production prices. In the observed test response, the seller identity and itinerary-specific booking URL were absent. The result therefore remains `liveMode: false`, is not a verified bookable flight, and cannot qualify as a verified-cheapest route. The optimizer unit test checks that test-mode offers are excluded from its verified-price result.

The timezone normalization now retains the original supplier local date-time and airport IANA timezone while also deriving UTC instants for schedule comparison. For example, the supplier's SFO local departure of `2026-12-11T07:44` is normalized to `2026-12-11T15:44:00Z`; the LHR local arrival remains `2026-12-12T03:48` and normalizes to `2026-12-12T03:48:00Z`.

Use a test-mode **read-write** token for offer-request creation, with `DUFFEL_FLIGHT_SEARCH_ENABLED=true` and `DUFFEL_ALLOW_TEST_MODE=true` only in a non-production environment. A production search requires an authorized live Duffel credential and account/market permissions; set `DUFFEL_ALLOW_TEST_MODE=false` and keep the production guard enabled. Production purchase access is not configured or implemented. Do not paste credentials into chat or commit `.env.local`.

The documentation describes capability, not this app's account entitlements, live inventory coverage, or route-level completeness. Verify those before enabling live search for users.
