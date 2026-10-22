# Chat, Boston transit and flight-search fixes — October 21, 2026

## Resolved

- Unknown city answers such as "I dont know" delegate city selection within the known destination area. A local handler selects major-city candidates without spending AI quota, while retaining dates, travelers, baggage, exclusions and required stops.
- "The south" narrows the comparison pool within an unambiguous supported country or region. For the US this includes Atlanta, Miami, Dallas, Houston, New Orleans, Nashville and Charlotte. Without clear scope, the chatbot asks one clarification instead of treating "south" as an airport or city.
- The candidate catalog now includes major cities in the US, Canada, Mexico and Japan alongside the existing European pool. These are curated candidate sets, not exhaustive worldwide city coverage. Exclusions and explicit mandatory cities still apply.
- Boston city searches resolve its US city center rather than confusing US/UK/Philippines results and multiple stations. Atlanta, Chicago, New York and Vancouver also have center references to distinguish same-named towns. Genuine unresolved ambiguity still requires clarification.
- Generic "Boston central station" routes from Boston's city center. The route displays the actual boarding station, operator, changes, and boarding/alighting distance from city centers when verified center coordinates are available. Distances are straight-line miles, not driving distances.
- Saved nonzero date flexibility no longer blocks flight searches with exact dates. The app searches stated dates first and explicitly discloses that adjacent flexible dates were not compared. Month-window searches retain their existing date comparison behavior. No trip preferences are silently overwritten.

## Live verification

- Current Vancouver → Boston → Atlanta → Chicago → Vancouver trip, January 18–30, 2027: five complete Google flight combinations appeared on the dashboard, with actual airline labels and middle legs. Returned totals started at USD 1,870 for three travelers; quotes can expire or change. Separate-ticket sums exclude unverified baggage.
- Boston → Atlanta, January 22, 2027: two bus itineraries in 2.89 seconds, including C&J, Peter Pan and FlixBus connections. First itinerary: 26h 59m, three changes. South Station was 0.5 miles from Boston center; Atlanta Bus Station 0.6 miles from Atlanta center.
- Boston → New York, October 25, 2026: no timetable results in 1.00 second despite correct city-center geocoding. This remains incomplete provider coverage, not evidence that services do not exist.
- Local intake endpoint: "I dont know" selected Boston/New York in 0.18 seconds; "the south" selected Atlanta/Miami in 0.03 seconds for a two-city US fixture. Neither repeated the city question.
- 341 regression tests across 52 files passed. TypeScript and changed-file lint checks passed.

No flights were booked, no paid Duffel data calls were made, and no reservations or payments were made. Transit schedules have no verified prices and remain excluded from totals.

