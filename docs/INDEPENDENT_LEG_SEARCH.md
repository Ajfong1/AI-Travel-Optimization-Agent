# Independent flight-leg search

The local app uses `FLIGHT_SEARCH_PROVIDER=fast_flights`. Every dated leg is searched independently as a one-way journey, with at most two concurrent Google requests per worker. Round trips include both directions; multi-city trips include all middle legs. Native Google multi-city mode is skipped by default. Duffel's cost block stays enabled. No paid integration, reservation or payment calls are made by this provider.

## Implementation

`scripts/fast_flights_compat.py` uses fast-flights 3.1.0's query builder and fetch integration API with a repaired parser. `src/server/providers/fast-flights/adapter.ts` connects it to the existing search/month-search/places endpoints. Local airport data resolves city names without supplier requests. The subprocess receives only OS runtime variables, without app secrets.

`scripts/flight-search-worker.py` uses a five-minute SQLite cache keyed by all query parameters, including dates, passenger categories, cabin and stops. No application-level city or independent-leg count cap; two concurrent requests per worker; three active workers maximum; 20-second request timeouts; process timeout scales with leg count from 100 seconds up to 30 minutes; cancellation kills the worker. No automatic retries. A layered combination search retains the five cheapest compatible paths per final option, avoiding exponential enumeration for larger trips.

Natural-language city counts such as “six cities instead of four” persist in `requestedCityCount`. Suggestions and permutation counts use that number, and existing searches become stale. The planner asks for confirmation of the revised draft. Six cities from the unrestricted 28-city Europe pool produce 271,252,800 possible city orders; this is a theoretical search space, not a claim that they have all been priced. Permutation indexing uses exact integer multiplication to handle large route counts.

Each dated leg must return a priced option with matching endpoints and departure date. Up to five returned choices per leg are combined and ranked, yielding at most five complete combinations. This does not guarantee globally cheapest inventory. Direct-only and maximum-connections filters apply to each leg. Separate one-way fares are summed in requested USD; baggage, airport-transfer costs and connection protection remain unverified and are visibly labelled. Baggage-inclusive offers are a separate comparison basis.

The parser handles loading shells, optional metadata, both best/remaining result groups, deduplication, unpriced rows, shortened clocks and airport changes. A missing middle or return leg cannot produce a complete total. Known airline names and flight numbers are retained. Unknown time zones are not invented; local strings are not marked UTC. Provider-supplied total journey durations support ranking. Resuming a city comparison with a different provider requires a fresh search.

## Airport lookup guardrails

Google-mode airport metadata comes from the local catalog. Those lookups have their own 240-per-minute bucket, separate from the unchanged 20-per-minute supplier quota. Previously they shared a bucket, so a city comparison could exhaust its own allowance before flight search. The client deduplicates lookups within a comparison and names the city, HTTP status or retry delay on errors. It requests fresh metadata so a corrected gateway or provider configuration takes effect immediately.

Alternate names such as Firenze/Florence, Milano/Milan and München/Munich are checked with the destination-country constraint. Catalog reference matches are marked explicitly without changing the airport's municipality. Nearby alternatives are not marked as exact city matches. Request fragments remain invalid destinations. Rome defaults to FCO rather than an alphabetical CIA selection; an explicit user-supplied airport overrides the default. This is still a resolved-airport search, not an exhaustive comparison of every airport in each metro area.

Failures now retain the affected dated flight leg and its safe diagnostic message in conditions. A corrected March 2027 six-city test returned five complete combinations with seven journeys; the original CIA-ending test had no priced options for LYS–CIA and CIA–SFO. The full corrected API result is in `work/flight-benchmark/airport-fix-six-city-result.json`. All 28 candidate cities were retested through the app's metadata endpoint (`airport-lookup-retest.json`).

## Live benchmark — October 20, 2026

All four original cases succeeded in all three uncached runs, with five complete options each. One adult, economy, fixed December 2026 dates, requested USD. Medians exclude Python startup:

| Trip | Median seconds | Lowest independent-fare sum |
|---|---:|---:|
| LAX–CDG–BRU–AMS–LAX, ten days | 2.496 | $1,082 |
| LAX–CDG–LAX, seven days | 1.488 | $753 |
| LAX–HND–LAX, thirty days | 1.706 | $1,424 |
| LAX–SFO–LAX, three days | 1.958 | $74 |

Full dates, samples and ranges: `work/flight-benchmark/independent-legs/report.md`. Quotes exclude unverified extras and are not confirmed bookable fares. This benchmark is not equivalent to the earlier outbound-only native result timings. It excludes ground-fare APIs, which lack an approved no-charge account.

The app's actual `/api/flights/search` endpoint also returned five complete options for the four-leg Europe route, with all four journey indexes preserved. Raw result: `work/flight-benchmark/independent-legs/app-result.json`.

## Run and deploy

Python packages are isolated in `work/flight-benchmark-venv/`. Recreate with Python 3.10+ and `scripts/requirements-fast-flights.txt`. The upstream release omits its required typing_extensions dependency; the requirements include it. The app defaults to that environment's Windows Python executable. Other hosts require `FAST_FLIGHTS_PYTHON` set to an absolute interpreter path and subprocess support. This is a local Node.js runtime integration, not an Edge/serverless deployment.

```powershell
& 'work/flight-benchmark-venv/Scripts/python.exe' scripts/search-multicity.py --leg LAX CDG 2026-12-03 --leg CDG BRU 2026-12-07 --leg BRU AMS 2026-12-10 --leg AMS LAX 2026-12-13
& 'work/flight-benchmark-venv/Scripts/python.exe' scripts/benchmark-independent-flights.py
& 'work/flight-benchmark-venv/Scripts/python.exe' -m unittest discover -s scripts -p test_fast_flights_compat.py -v
```

The CLI supports `--direct-only`; `--native-only` explicitly enables native mode for diagnostics and disables independent fares. CLI results do not change saved dashboard trips. Ground research: [Transit search options](TRANSIT_SEARCH_OPTIONS.md).
