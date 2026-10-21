# fast-flights multi-city repair

**Superseded by independent-leg search:** the app now skips native multi-city requests and searches every leg independently. See [current implementation](INDEPENDENT_LEG_SEARCH.md). The native-first behavior and measurements below are the historical diagnostic stage.

The isolated Python search adapter now searches all dated multi-city legs, using native multi-city mode first and an explicitly labelled independent-one-way fallback. The dashboard provider has not been switched; Duffel remains cost-blocked. No paid integrations, account credentials, booking or reservation calls are used by these scripts.

## Diagnosis

The original four-leg query correctly used `trip="multi-city"`. Google returned the complete search form with `Loading results`, empty result sections, and alliance metadata without airline metadata. Upstream fast-flights 3.1.0 indexed `payload[7][1][1]` unconditionally and raised `IndexError`. This was an incomplete search response, not proof of no available flights.

Tested a second native route (LAX–CDG–FCO–LAX), the `tfu` all-results parameter, additional protobuf query flags, the public page's complete batch search request, and the read-only shopping-results endpoint. None returned native complete multi-city results. These experiments are diagnostic scripts, not runtime dependencies. The underlying native combined-fare limitation remains unresolved.

## Implemented behavior

`scripts/fast_flights_compat.py` reuses the project's query builder and fetch integration API, with a compatibility parser and bounded requests:

- Recognizes missing result data and returns a meaningful incomplete-response status instead of crashing on optional metadata.
- Reads both Google's best and remaining result groups, deduplicates identical rows and excludes unpriced options from price ranking.
- Handles shortened time arrays and preserves dates, actual airline names, airport names/codes, flight numbers and durations.
- Preserves and flags airport changes, such as LCY–LHR, rather than rejecting all flights in that response.
- Sends the complete dated native multi-city request first. A first-leg quote cannot count as a complete itinerary.
- If native results are incomplete, searches every leg as a one-way query, with at most two concurrent requests and a 20-second timeout per request, without automatic retries. At most seven requests are made for a six-leg search.
- Checks returned endpoints/dates, respects direct-only filtering, and requires a priced option for every leg before producing any complete trip option.
- Ranks up to five complete combinations from the five cheapest returned choices per leg. Does not claim exhaustive or globally cheapest coverage.
- Never labels the summed one-way prices as a single multi-city ticket. Separate tickets, baggage, airport transfers and protection require independent review. No reservations are created.

## Live verification on October 20, 2026

One adult, economy, requested USD. Fixed December 2026 dates. Times include the native attempt and all fallback searches, excluding Python startup. These are individual observations, not performance guarantees.

| Route | Time | Returned complete options | Lowest separate-fare sum |
|---|---:|---:|---:|
| LAX → CDG Dec 3 → BRU Dec 7 → AMS Dec 10 → LAX Dec 13 | 3.663 s | 5 | $1,082 |
| LAX → CDG Dec 3 → FCO Dec 7 → LAX Dec 13 | 4.090 s | 5 | $767 |

Both used the independent-one-way fallback. Quotes are search estimates, not verified bookable offers or confirmed totals including baggage. Full segments, individual leg prices, transfer flags and limitations are saved in `work/flight-benchmark/multicity-fixed/original.json` and `control.json`. No Duffel or paid provider calls were made.

## Run another search

From the project directory in PowerShell:

```powershell
& 'work/flight-benchmark-venv/Scripts/python.exe' scripts/search-multicity.py --leg LAX CDG 2026-12-03 --leg CDG BRU 2026-12-07 --leg BRU AMS 2026-12-10 --leg AMS LAX 2026-12-13
```

Use `--direct-only` for nonstop legs, or `--native-only` to disable the separate-ticket fallback. The JSON includes the actual source and an explicit incomplete status if any leg fails. Custom output paths use `--output`. This CLI does not alter saved dashboard trips.

## Validation and dependencies

Run regression tests without network access:

```powershell
& 'work/flight-benchmark-venv/Scripts/python.exe' -m unittest discover -s scripts -p test_fast_flights_compat.py -v
```

The ten tests cover native mode/leg preservation, all middle legs, missing legs, first-leg-only prices, complete native responses, direct-only filters, empty metadata/loading shells, both result groups, missing prices, abbreviated clocks, malformed segments, deduplication and airport transfers.

Dependencies installed in the isolated virtual environment: fast-flights 3.1.0, primp 2.0.1, protobuf 7.36.2, selectolax 1.0.1, typing_extensions 4.16.0. The upstream release omits the required typing_extensions dependency, so it was installed explicitly. Project runtime packages were not modified.
