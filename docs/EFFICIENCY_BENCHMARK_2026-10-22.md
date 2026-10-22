# Agent efficiency review — October 22, 2026

## Measured chatbot changes

The app now handles unambiguous answers to an active question locally: confirmations of a known pending value, city counts, moving pace, explicit duration units, date flexibility, an explicit adult party, one unambiguous child/infant age with units, and named bag counts. Flexible city delegation and bare bag counts already had local handling. Compound requests and ambiguous replies still reach the configured conversational provider. The same shared field schemas and readiness rules validate the result.

The original four trip types were used as prepared drafts for a single confirmation turn each. These are real HTTP calls to the running local development server, not mocked provider timings or timings of an entire conversation. Baseline: one sample per case, four Gemini calls. Revised: three warm samples per case, zero model calls. All twelve revised replies confirmed the requested value and were search-ready. The benchmark does not create saved trips or launch flights.

| Draft | Baseline confirmation | Baseline outcome | Revised warm median (range) |
| --- | ---: | --- | ---: |
| Europe: Paris–Brussels–Amsterdam | 15.982 s | Confirmed | 25.4 ms (23.2–327.8) |
| One city: Paris | 29.324 s | invalid_model_response | 27.4 ms (23.4–28.0) |
| Long trip: Tokyo | 3.625 s | model_unavailable | 25.8 ms (23.4–33.4) |
| Short trip: San Francisco | 0.511 s | model_unavailable | 27.1 ms (25.4–34.6) |

Development compilation matters: the first revised request after a code edit took 3.119 s. Initial module loading, browser rendering, dashboard navigation and network/model variability are not removed by the warm results. Three baseline errors are failures, not successful completions. This small sample is not a provider reliability estimate or a guarantee that free-form chat is instant. No additional Gemini request is spent on local answers.

## Airport and route setup

Airport search data is normalized once per server module. Exact IATA points use a map; nearby candidates use a bounded 256-entry cache keyed by normalized query, country and result limit. Cached results are copied before returning so callers cannot modify later results. Only static airport data is cached here, never fares or traveler information. City comparisons resolve airports in groups of up to four, retain travel order, and reuse resolved codes within a batch. Fare requests and their existing budgets remain sequential.

| Lookup set | Baseline median | Revised warm median |
| --- | ---: | ---: |
| Europe: Paris–Brussels–Amsterdam + LAX map point | 97.277 ms | 0.055 ms |
| One city: Paris + LAX map point | 35.519 ms | 0.029 ms |
| Long trip: Tokyo + LAX map point | 31.568 ms | 0.018 ms |
| Short trip: San Francisco + LAX map point | 35.556 ms | 0.023 ms |

Each baseline set has 20 samples; each revised set has 60. Module startup is excluded. Revised measurements include initial misses as well as repeated hits, but their median describes warm reuse. Lookup sets contain three destination cities for multi-city and one for other cases. The catalog index reduces repeated work; it does not change airport eligibility, country filtering, distance semantics or supplier inventory.

Fresh validated planner responses now queue the dashboard handoff directly, removing one redundant readiness HTTP request. Manual searches of saved drafts still run the local repair/readiness check. Validation at flight endpoints and the one-time handoff queue are retained.

The review also fixed departure parsing that could absorb ‘back’ into ‘LA, back to LA’ or cross a sentence boundary. Comma-separated airport alternatives remain supported. A stale six-city restriction in the model instructions was removed; the existing schema still accepts positive counts without the former six-city application cap.

## Original four flight searches

Each trip used the production Python worker with one adult, economy, and every leg searched independently. One first pass and one warm repeat per case; Python process startup is included. Google’s public search was used, with no paid provider, booking or payment. Existing unexpired cache entries were not cleared, so the first pass is not claimed as a controlled cold-cache benchmark. This is a separate end-to-end flight smoke benchmark, not a before/after measurement of the new chatbot code.

| Trip / exact December 2026 dates | Legs | First pass | Warm repeat | Complete options, both runs |
| --- | ---: | ---: | ---: | ---: |
| Europe: Paris–Brussels–Amsterdam: Dec 3 LAX–CDG; Dec 7 CDG–BRU; Dec 10 BRU–AMS; Dec 13 AMS–LAX | 4 | 3.909 s | 0.746 s | 5 / 5 |
| One city: Paris: Dec 3 LAX–CDG; Dec 10 CDG–LAX | 2 | 2.421 s | 0.750 s | 5 / 5 |
| Long trip: Tokyo: Dec 1 LAX–HND; Dec 31 HND–LAX | 2 | 2.717 s | 0.763 s | 5 / 5 |
| Short trip: San Francisco: Dec 4 LAX–SFO; Dec 7 SFO–LAX | 2 | 2.972 s | 0.740 s | 5 / 5 |

All requested legs returned priced choices in these eight runs. The five-minute fare cache already existed; warm-repeat speed is not credited as a new optimization. These checks do not establish fare completeness, baggage inclusion, protected connections, global cheapest coverage or current availability. Regional alternatives remain bounded batches, rather than launching every permutation.

## Reproduce from the complete local application checkout

Start the development server first. The baseline file records the earlier code; rerunning it against the revised code cannot reconstruct old behavior. The following commands measure the revised paths and make public flight requests:

```powershell
$env:AGENT_BENCHMARK='revised-warm'
$env:AGENT_BENCHMARK_REPEATS='3'
npm test -- src/server/conversation/efficiency-benchmark.test.ts
Remove-Item Env:AGENT_BENCHMARK,Env:AGENT_BENCHMARK_REPEATS
& 'work/flight-benchmark-venv/Scripts/python.exe' scripts/benchmark-agent-flights.py
node scripts/report-agent-efficiency.mjs
```

The HTTP benchmark is opt-in and skipped in the normal test suite. Never enable it against a remote or shared production endpoint. Raw timing JSON stays local in work/efficiency-benchmark. This GitHub repository remains a documentation publication; these local source files and scripts are not included in a docs-only clone.
## Final validation

391 TypeScript regression tests passed across 55 active files; the opt-in HTTP benchmark is skipped during ordinary runs. The benchmark itself passed separately for all twelve warm confirmations. All 13 Python compatibility tests, TypeScript checking, and ESLint passed. Cache-copy isolation, country/limit keys, bounded airport concurrency, preserved travel order, compound-message fallback, age units, personal-item defaults, and departure/return parsing have regression coverage.
