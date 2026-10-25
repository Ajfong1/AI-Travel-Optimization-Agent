# Asian city, lodging and one-way search fixes

Taipei previously failed before a lodging request: it had no Xotelo destination mapping. The first hotel-heavy listing pages could also hide hostels, and property keys belonging to city districts were discarded. Flight readiness required a return date even for one-way journeys.

## Changes

- Shared city references cover Taipei, Manila, Tokyo, Osaka, Seoul, Bangkok, Singapore, Hong Kong, Kuala Lumpur, Hanoi, Ho Chi Minh City and Cebu City. Qualified names and common aliases share the same city, country, major airport and lodging mapping. Explicit airport codes remain valid choices.
- Taipei defaults to TPE, Manila to MNL, Seoul to ICN and Osaka to KIX. Ontario resolves to ONT in the existing airport catalog. This does not mean nearby airports are all priced automatically.
- Lodging discovery continues through up to 1,000 cached listings when a requested type is scarce. District property keys are accepted after the location check. Asian properties are checked against city coordinates rather than a possibly distant flight gateway.
- Only requested hotel/hostel types are priced. Up to six different properties per type are checked until three quotes return. Hotel and hostel workers run concurrently, with at most two quote requests per search. Rate requests time out after 12 seconds; list requests after 15 seconds. Failed/empty quotes can move on to another property. Both requested types retain a place in the five-result shortlist when quotes exist.
- Empty samples, rate errors and incomplete flight searches are identified as coverage/pricing failures rather than proof that a city has no hotels or flights. No paid fallback is called.
- Explicit one-way intent is stored, validated, understood by local/model intake and edits, and selectable under Flight type on the dashboard. A return date is not required. A one-way month search compares departures without inventing a stay length or return leg. Return searches preserve the visited city when switching flight type.
- One-way accommodation uses actual local arrival plus a stated stay length or check-out date. With no stay dates, the UI asks for them rather than inventing a return flight. The dashboard labels the journey as One-way.
- Taipei transit starts at the city center and lets the router select boarding stops. Station distances remain disclosed when schedules return. Taiwan operator links replace irrelevant European links for Taiwan searches.
- A follow-up hosted check caught a legacy Manila trip incorrectly sent through regional city planning because the airport municipality differs from the destination. A confirmed single-city endpoint now repairs that stale mode without asking how often to move cities. Child-only requests are also stopped explicitly instead of silently adding an adult to the Google query; traveler corrections retain known baggage.

## Free live checks

Flight checks used public Google searches via the existing isolated fast-flights worker, one adult, economy, up to two connections. Each case returned five complete options. These were search quotes, not reservations or verified baggage-inclusive fares.

| Case | Dates | Search seconds |
| --- | --- | ---: |
| ONT → MNL → ONT | October 30 / December 31, 2026 | 1.570 |
| ONT → MNL, one-way | October 30, 2026 | 1.199 |
| LAX → TPE | October 30, 2026 | 2.005 |
| TPE → MNL | November 4, 2026 | 0.857 |
| MNL → HND | November 8, 2026 | 1.113 |
| ICN → BKK | November 12, 2026 | 1.859 |
| BKK → SIN | November 15, 2026 | 1.610 |

Lodging checks used November 10–13, 2026, two adults, one room, USD, both requested types. Counts below are the final five displayed results, with type classification supplied by Xotelo. All six returned dated quotes for both types.

| City | Hotels | Hostels | Search seconds | Listings sampled |
| --- | ---: | ---: | ---: | ---: |
| Taipei | 2 | 3 | 38.815 | 200 |
| Manila | 3 | 2 | 12.788 | 100 |
| Tokyo | 3 | 2 | 29.222 | 400 |
| Seoul | 3 | 2 | 25.914 | 200 |
| Bangkok | 3 | 2 | 17.907 | 300 |
| Singapore | 2 | 3 | 15.146 | 200 |

The six other city lodging keys were checked against small public listing samples and their geographic locations. Airport resolution was checked for all twelve cities and aliases. Taipei → Taichung on October 30 returned three transit schedules from city names alone, without a station-selection question. Timetable coverage varies; no train/bus fares were invented.

## Verification and limits

Regression tests cover one-way readiness, month search, model omissions, changing flight type, return itineraries, one-way stay dates, district lodging keys, hostel discovery after hotel-only pages, empty-rate fallback, airport aliases and Taipei city-center routing. TypeScript and ESLint check the changed source. Browser checks use a separate guest-only local server with account syncing and paid flight access disabled.

Free search services can be slow or unavailable, and cold hosted starts add latency. These bounded samples do not establish city-wide lowest prices or room capacity. Supplier taxes, bed allocation, baggage and availability still need confirmation. No flight orders, bookings, payments or paid travel-provider calls were made.

Sources: [Xotelo endpoint documentation](https://xotelo.com/), [Taipei destination identifier](https://www.tripadvisor.com/Hotels-g293913-Taipei-Hotels.html), [Taiwan Railway city/station search](https://www.railway.gov.tw/tra-tip-web/tip?lang=EN_US).
