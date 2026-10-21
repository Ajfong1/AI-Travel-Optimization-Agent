# Four-trip end-to-end check

Ran the original four date/route fixtures through the browser chatbot, explicit flight search, saved dashboard comparison and chosen-trip selection. One adult, economy, connections allowed, zero checked/carry-on bags. Searches used the configured fast-flights provider; no Duffel live requests, bookings or payments were made. Accommodation search was not triggered.

| Case | Dates (2026) | Route | Complete combinations | Lowest returned USD quote |
| --- | --- | --- | --- | --- |
| Multi-city | December 3–13 | LAX–CDG–BRU–AMS–LAX | 5 | $1,081 |
| One city | December 3–10 | LAX–CDG–LAX | 5 | $753 |
| Long trip | December 1–31 | LAX–HND–LAX | 5 | $1,423 |
| Short trip | December 4–7 | LAX–SFO–LAX | 5 | $74 |

These are observed search quotes, summed from separate one-way tickets, excluding unverified baggage costs. They are not guarantees of availability or the global cheapest fare. The old fixture labels used date differences: these cases span 10, 7, 30 and 3 nights respectively, or 11, 8, 31 and 4 calendar days. Overnight flight arrivals reduce actual hotel nights.

## Smooth

- All four cases returned five complete combinations, with actual airline names and expandable journey details.
- The multi-city quote included every middle leg, not just the international flights.
- Flight options persisted when opening the dashboard and switching saved trips. Chosen selections updated planning totals and schedules.
- Tokyo and San Francisco intake each reached search readiness from one conversational prompt.
- In the final replay, both formerly problematic prompts reached search readiness with no questions or contradictions: multi-city intake took 2.64 seconds; Paris took 5.86 seconds. These are single-run intake times, not flight-search averages.

## Issues corrected

- Asking for pace despite an already confirmed fixed route, then clearing that route on a pace answer.
- Incorrect model arithmetic creating a duration/date contradiction when the user supplied dates without a duration. The server now derives nights in this situation.
- Confusing the visited city with a round trip's final endpoint, and omitting a one-city visit from structured updates. Strengthened conversational instructions and added a server readiness check for a missing round-trip stop.
- Optional inferred preferences or extra preference questions blocking a complete flight search.
- Missing origin/endpoint confirmation questions that could leave a draft without a usable next step.
- First-search button saying “Search again”; it now says “Find flights.” Search status also distinguishes airport resolution from flight search.
- Collapsed offers with identical prices looking indistinguishable. Summaries now include departure time, flying time and layover count.
- Dashboard headings showing only Los Angeles to Los Angeles. They now include the visited route.
- “Between cities” map including connection airports such as JFK, LIN and DUB. It now connects CDG–BRU–AMS; full route keeps flight segments.
- An expired selected quote staying expired after an identical offer was refreshed. Matching provider, mode, ID, price and itinerary can refresh that selection; changed offers require a new selection.

## Limits

The model is probabilistic; validation and regression tests cover the failures observed here, but these checks cannot guarantee every phrasing. Quotes expire and need refresh. Rail/bus live fare credentials remain unconfigured; no rail/bus fare coverage is claimed by this check. Existing local provider blocks and conversational request limits remain in force.

Validation: 299 tests across 44 files passed, plus TypeScript, ESLint and whitespace checks. Browser verification confirmed the refreshed selected fare, planning total and CDG–BRU–AMS map. Screenshot: `work/browser-check/four-trip-flow-fixed.png`.
