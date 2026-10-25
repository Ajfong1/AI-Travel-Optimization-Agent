# Current implementation

Snapshot: **October 24, 2026**. Complete source, tests and scripts are saved in the private [wayfarer-hosting repository](https://github.com/Ajfong1/wayfarer-hosting). The public AI-Travel-Optimization-Agent repository publishes the documentation. Earlier audits remain historical records.

## Runtime and conversation

The application uses Next.js App Router, React, TypeScript, Tailwind CSS, Zod and Vitest. The Render Free Docker service runs Next.js and the Python flight worker together. Supabase supplies email verification, password sign-in and account-owned trip workspaces; Brevo supplies custom SMTP. The user confirmed password sign-in worked. Free-service quotas, sleep and email delays remain limitations.

Guest trips remain in browser storage. Signed-in users have personal workspaces and account sync, with database access limited to their own row by Supabase policies. Existing browser trips require an explicit import choice. The planner-to-dashboard search handoff uses a one-time session-storage queue.

The planner posts to `/api/intake`. A server dispatcher selects Gemini, Groq or deterministic parsing. Model proposals undergo field-schema validation and shared date, route, party and baggage checks. Errors preserve the draft; keys stay server-side. Provider errors do not trigger automatic model/provider fallback.

The UI shows one active question. Uncertainty about cities or pace resolves locally as a flexible choice. Unambiguous confirmations, city counts, moving pace, explicit durations, date flexibility, simple adult counts, a single pending age with units and bag counts also avoid a model request. Compound and ambiguous messages retain model interpretation. A search-ready response saves the trip, uses its fresh server validation, queues a one-time search, shows a loading indicator and opens the dashboard Transport section. Loading an old draft does not automatically repeat a search.

## Flights

The `fast_flights` provider uses a Python worker and the upstream fast-flights query/fetch integration with compatibility parsing. Airport resolution uses a local catalog, normalized aliases and major-airport gateways. Banff uses Calgary; Santa Monica resolves through Los Angeles-area airports. Twelve Asian destinations have shared airport and lodging references. Explicit airport codes remain valid; a gateway match does not prove every nearby airport was priced.

Every dated journey is searched independently, including middle and return legs. One-way trips do not require a return date. Missing priced legs cannot become a complete itinerary total. Child-only requests are stopped explicitly rather than silently adding an adult fare.

The worker has no application-level city or independent-leg count cap, with a five-minute SQLite cache, bounded concurrency/timeouts, cancellation and no automatic retries. Up to five returned options per leg feed bounded combination ranking. These are separate-ticket sums, excluding unverified extras; they do not prove protected connections, ticketability or global optimality.

Flexible months compare fitting date combinations. Regional comparisons enumerate scoped city orders in batches, applying exclusions and required cities. Hosted month scheduling respects the free container's single-worker capacity and stops on contention instead of counting unsearched dates as unavailable. Large searches can require substantial requests and time. Partial coverage remains visible.

Duffel adapters and historical test-mode diagnostics exist separately. Hosted startup disables Duffel flight search and blocks live Duffel data. Stored credentials do not authorize purchases or paid data requests. No booking, reservation or payment flow is implemented.

## Dashboard and saved plans

The current defaults use key-free Xotelo lodging and Transitous schedules; see [Free providers](FREE_PROVIDERS.md). Previous RapidAPI/Tictactrip connections are opt-in, with no paid fallback. Schedule fares remain unknown and excluded from totals. Transit starts from city names and coordinates, allowing the router to choose boarding stops and report available station distances. Taipei uses city-center routing and Taiwan operator links. The mixed-trip builder includes timing and overnight travel but does not invent fares.

Features include a teal visual theme, SVG route mark, compact section navigation, city comparisons, expandable flight/layover details and per-journey prices, direct-only filtering, chosen transport, ordered route mapping, schedules, accommodation selections and estimated totals by currency. Saved trips support filtering, deletion and recovery. Signed-in workspaces sync; guest browser storage is not a backup.

Xotelo city mapping includes Banff, Vancouver and major Asian destinations. Discovery can inspect up to 1,000 cached listings when a requested type is scarce; district properties require a location check. An empty sample or pricing error is not evidence that a city has no accommodation. Only requested hotel/hostel types are priced.

Stays of 28 nights or more are split into contiguous periods of at most 21 nights to respect Xotelo's date-range limit. A full-stay estimate requires fresh quotes for every period, with matching property, party, room count and currency. The adapter sums actual dated prices and never extrapolates a missing period. Rate checks are limited to 20 per city. The UI shows the separate periods and warns that room changes may be required. Xotelo does not verify bed layouts, room capacity, taxes or continuous room availability.

The DataCrawler/RapidAPI lodging adapter uses `booking-com15.p.rapidapi.com`, separately from official Booking.com Demand API access. It searches selected city stay dates, filters hotels/hostels, preserves quote sources/times, and adds selected stays to the estimated total. Property-to-outgoing-airport distance is straight-line distance when coordinates exist.

Tictactrip requires approved partner access and explicit configuration. Rail Europe ERA, Hostelworld and other researched providers are not connected merely because documentation exists. Airbnb has external search links rather than live inventory. Activities have domain/planning logic rather than demonstrated live inventory.

## Recent verification

| Check | Latest recorded result |
| --- | --- |
| Full Vitest suite, Asia/legacy-trip revision | 472 tests passed; one opt-in benchmark skipped |
| Later long-stay revision | 51 focused lodging, saved-plan, room and one-way tests passed |
| TypeScript | `tsc --noEmit` passed |
| ESLint | Changed functional source passed targeted checks |
| Git whitespace | `git diff --check` passed; Windows line-ending notices were informational |
| Gemini | Initial trip and infant-age follow-up worked; a temporary provider failure preserved the draft |
| Automatic search browser flow | A complete November 5–7, 2026 LAX–SFO–LAX trip opened the dashboard without pressing Search flights and returned five options |
| Efficiency review | Warm confirmation medians 25–28 ms; four flight cases each returned five complete options in both first and cached passes. See [benchmark report](EFFICIENCY_BENCHMARK_2026-10-22.md). |
| Earlier four-trip run | Multi-city, one-city, long and short cases returned complete options; see the dated report |
| Free Asia flight checks | Seven cases each returned five complete options; 0.857–2.005 seconds locally |
| Free Asia lodging checks | Six cities returned both hotels and hostels; 12.788–38.815 seconds locally |
| Taipei → Taichung | Three schedules returned from city names without requiring station knowledge |
| Manila, October 31–December 5, 2026 | 35-night check returned dated hotel and hostel quotes in 29.16 seconds; hosted UI showed three hotels and two hostels with period breakdowns |
| Hosting and accounts | Linux Docker deployment succeeded; user confirmed password login |

These are dated development and hosted observations, not a production certification. No booking/payment or paid travel-provider request was performed during these checks. Historical prices are snapshots, not current offers. Repository publication does not rerun supplier searches or publish local test artifacts.

## Remaining limits

- No booking/payment flow. Hosting, accounts and sync are implemented, but free-service sleep, quotas and email delays remain limitations.
- No guarantee of exhaustive inventory, baggage-inclusive final prices, globally cheapest routes or continued availability.
- Partner APIs need separate authorization; configuration does not prove connectivity.
- Google/RapidAPI quotas and subscription terms remain authoritative. Local caps cannot prove free-tier billing status.
- Browser checks are development automation, not comprehensive accessibility, security, load or cross-browser certification.

See the [repository snapshot](REPOSITORY_SNAPSHOT_2026-10-24.md) for included source and excluded local data.
