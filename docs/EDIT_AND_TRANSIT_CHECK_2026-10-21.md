# Edit and transit debugging check

Development check on October 21, 2026. No reservations, payments, paid supplier searches or new subscriptions were used. Chat-assisted edits use the already configured conversation provider; its quotas still apply.

## Findings and fixes

- The dashboard's Edit trip endpoint used a small deterministic interpreter even though the main planner supported conversational input. Natural city/date/budget changes were often unsupported. Common origin, natural-date and budget edits now run locally; other edits use the configured conversational provider, with atomic validation, mandatory-stop protection, clarification history and safe error messages.
- After applying an edit, the dashboard now rechecks readiness using the local repair mode instead of spending another model request for city-count updates. Date/route changes invalidate affected data. Confirming the same values or reversing an edit now reconnects semantically compatible quotes/selections without extending quote expiry.
- Ground queries previously requested only the first few combined-mode results, potentially hiding rail or bus alternatives. Rail and bus now have separate bounded queries (up to three results each), then deduplication. Ambiguous Salzburg lookup is constrained to Austria. Route criteria survive reload and changing them marks previous results stale.
- Loading feedback, a client timeout, required departure date, named city headings, operator summaries, and official operator links make the result state visible. A response with no schedules is now incomplete coverage, not a successful empty inventory claim. No fares are inferred or added as zero-cost transport.

## Ground benchmark

Requests used the app's `/api/ground/plan` endpoint. These are single observed local response times after the fixes, with city-lookup caches already warm; they are not load-test or latency guarantees. The first patched request may include compilation overhead.

| Case | Date | Returned schedules | Response seconds | Coverage |
| --- | --- | ---: | ---: | --- |
| Vienna → Salzburg (Austria) | October 25, 2026 | 3 | 2.74 | Bus/coach routes including FlixBus; the feed did not return rail for this query |
| Budapest → Győr (Hungary) | October 25, 2026 | 6 | 0.66 | Rail and coach, including MÁV, GYSEV and FlixBus operators |
| Vienna → Budapest (cross-border) | October 25, 2026 | 6 | 0.63 | Rail and coach, including MÁV and FlixBus |
| Vienna → Budapest (requested future-date case) | December 4, 2026 | 0 | 0.64 | Incomplete free-feed coverage; explicit operator links provided |

The December query also returned no itineraries when sent directly to the public service, before app normalization. The free Deutsche Bahn wrapper was tested as an alternative and still returned HTTP 503. A currently available free feed was not demonstrated for that date; the app does not substitute a near-term schedule or present it as December availability.

ÖBB confirms [Vienna–Budapest train services](https://www.oebb.at/en/reiseplanung-services/oebb-zugverbindungen/zugverbindungen-europa/budapest), including [Railjet/EuroCity service](https://www.oebb.at/en/tickets-kundenkarten/oesterreich-europa/sparschiene/sparschiene-europa/ungarn). FlixBus publishes its [Vienna–Budapest route](https://www.flixbus.com/bus-routes/bus-vienna-budapest). Operator links are manual searches, not API-verified departures or ticket quotes for a selected date. Transitous's [data documentation](https://transitous.org/doc/) explains its dependence on supplied schedule feeds.

## Edit checks

Seven independent edit scenarios against a synthetic Paris trip all returned `applied` in the final local API check: ISO departure date, natural-language return date, departure city, direct-only preference, hostel-only preference, budget, and adding Budapest after Paris. Warm common edits took approximately 0.03–0.04 seconds; the conversational city-route edit took 1.37 seconds. The initial request in that check took 9.42 seconds including development-server compilation. A temporary model-service failure was reproduced during earlier checks; ordinary date/origin/budget edits no longer depend on that service, while provider failures for conversational edits still preserve the draft and require manual retry.

The browser also verified applying and reversing the connection preference, preserving compatible selected transport/lodging and the existing planning total, then displaying six Vienna–Budapest schedule alternatives. Benchmark ground dates were separate from the trip's December requirements.

## Airbnb

The accommodation section now provides an external Airbnb search for each selected city stay, including dates and adult/child/infant counts. There is no API charge or automated listing retrieval; the user opens Airbnb to compare availability and prices. Airbnb prices are not imported into totals.

Airbnb's [software partner page](https://www.airbnb.com/software-partners) and [API terms](https://www.airbnb.com/help/article/3418) describe partner-controlled access and authorized scopes. No verified free guest-listing API was established for this app. Adding a live Airbnb quote provider remains dependent on suitable access and verified pricing semantics.

Raw responses, benchmark scripts and screenshots stay local under `work/`; they are not part of the public documentation upload.
