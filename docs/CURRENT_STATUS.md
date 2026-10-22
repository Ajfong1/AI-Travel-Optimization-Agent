# Current implementation

Snapshot: **October 22, 2026**. This describes the local application and supersedes outdated feature-status statements in the original audits. The initial GitHub publication contains documentation rather than application source.

## Runtime and conversation

The application uses Next.js App Router, React, TypeScript, Tailwind CSS, Zod, Vitest, and Drizzle/PostgreSQL schema definitions. Current trip persistence is browser local storage, with a session-storage queue for the one-time planner-to-dashboard search handoff. Database-backed trip persistence, production authentication and multi-user ownership are not demonstrated.

The planner posts to `/api/intake`. A server dispatcher selects Gemini, Groq or deterministic parsing. Model proposals undergo field-schema validation and shared date, route, party and baggage checks. Errors preserve the draft; keys stay server-side. Provider errors do not trigger automatic model/provider fallback.

The UI shows one active question. Uncertainty about cities or pace resolves locally as a flexible choice. Unambiguous confirmations, city counts, moving pace, explicit durations, date flexibility, simple adult counts, a single pending age with units and bag counts also avoid a model request. Compound and ambiguous messages retain model interpretation. A search-ready response saves the trip, uses its fresh server validation, queues a one-time search, shows a loading indicator and opens the dashboard Transport section. Loading an old draft does not automatically repeat a search.

## Flights

The local `fast_flights` provider uses a Python worker and the upstream fast-flights query/fetch integration with compatibility parsing. Airport resolution uses a local catalog. Every dated journey is searched independently, including middle and return legs. Missing priced legs cannot become a complete itinerary total.

The worker has no application-level city or independent-leg count cap, with a five-minute SQLite cache, bounded concurrency/timeouts, cancellation and no automatic retries. Up to five returned options per leg feed bounded combination ranking. These are separate-ticket sums, excluding unverified extras; they do not prove protected connections, ticketability or global optimality.

Flexible months compare fitting date combinations. Regional comparisons enumerate scoped city orders in batches, applying exclusions and required cities. Large searches can require substantial requests and time. Partial coverage remains visible.

Duffel adapters and historical test-mode diagnostics exist separately. The no-charge local configuration keeps live Duffel requests blocked. Stored live credentials do not authorize purchases or paid data requests.

## Dashboard and saved plans

The current defaults now use free Xotelo lodging and Transitous schedules; see [Free providers](FREE_PROVIDERS.md). Live prototype checks returned three Porto hostel quotes, three Milan hotel quotes, and two Paris–Brussels schedule options. Previous RapidAPI/Tictactrip connections are opt-in, with no paid fallback. Schedule fares remain unknown and excluded from totals.

Features include section navigation, compact city comparisons, expandable flight cards, direct-only filtering, chosen transport, ordered route mapping, schedules, accommodation selections and estimated totals by currency. Saved trips support filtering, deletion and recovery. Browser storage is not cloud sync or a backup.

The DataCrawler/RapidAPI lodging adapter uses `booking-com15.p.rapidapi.com`, separately from official Booking.com Demand API access. It searches selected city stay dates, filters hotels/hostels, preserves quote sources/times, and adds selected stays to the estimated total. Property-to-outgoing-airport distance is straight-line distance when coordinates exist.

Tictactrip requires approved partner access and explicit configuration. Rail Europe ERA, Hostelworld and other researched providers are not connected merely because documentation exists. Activities have domain/planning logic rather than demonstrated live inventory.

## Recent verification

| Check | Latest recorded result |
| --- | --- |
| Vitest | 391 tests across 55 active files passed on October 22, 2026; one opt-in HTTP benchmark skipped |
| TypeScript | `tsc --noEmit` passed |
| ESLint | Changed automatic-search and conversation files passed |
| Git whitespace | `git diff --check` passed; Windows line-ending notices were informational |
| Gemini | Initial trip and infant-age follow-up worked; a temporary provider failure preserved the draft |
| Automatic search browser flow | A complete November 5–7, 2026 LAX–SFO–LAX trip opened the dashboard without pressing Search flights and returned five options |
| Efficiency review | Warm confirmation medians 25–28 ms; four flight cases each returned five complete options in both first and cached passes. See [benchmark report](EFFICIENCY_BENCHMARK_2026-10-22.md). |
| Earlier four-trip run | Multi-city, one-city, long and short cases returned complete options; see the dated report |

These are development observations, not a production certification. No booking/payment was performed. Historical prices are snapshots, not current offers. Publishing docs does not rerun supplier searches or publish local test artifacts.

## Remaining limits

- No booking/payment flow, demonstrated production deployment, user accounts, cloud sync or verified database-backed UI persistence.
- No guarantee of exhaustive inventory, baggage-inclusive final prices, globally cheapest routes or continued availability.
- Partner APIs need separate authorization; configuration does not prove connectivity.
- Google/RapidAPI quotas and subscription terms remain authoritative. Local caps cannot prove free-tier billing status.
- Browser checks are development automation, not comprehensive accessibility, security, load or cross-browser certification.
