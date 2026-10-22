# Mixed transport planning — October 21, 2026

The dashboard now has a **Mixed trip** tab under Transport options. Start with a searched city itinerary, expand a leg, and choose its flight, train or bus. Search ground routes one leg at a time with **Find trains & buses** or **Evening departures**. The date field allows an explicit different departure date, including a previous evening; the app does not silently shift dates.

## Cost and daytime comparison

- **Lowest priced combination** compares the loaded, compatible legs with known prices. Unpriced schedules are excluded. It uses a bounded 64-path beam search and does not guarantee a global optimum over all services or city orders.
- **More daytime in cities** minimizes estimated travel during 09:00–18:00, based on endpoint local clocks. Estimates include two hours before and one hour after flying, or 30 minutes at each end for ground travel. Time-zone changes can affect the estimate. It is a planning proxy, not an exact measure of free sightseeing time.
- Transitous evening searches start at 18:00 in the origin's provider-supplied time zone, or 18:00 UTC if unavailable. Only validated local times receive overnight labels. No sleeper berth, sleep quality, protected connection, hotel saving or overnight availability is inferred.
- Full one-way flight alternatives are retained per leg separately from the five cheapest complete flight combinations. This allows the planner to consider a more expensive but quicker leg even when it was absent from the complete-trip shortlist. These results remain limited by the supplier's returned options.
- Each entered operator ticket total is explicitly labelled as an **entered fare**, not a live quote. Enter the total for all travelers in the displayed currency. It can be included in planning comparisons, with its provenance retained in the chosen trip and budget.
- With missing ground fares, the displayed amount is a **known subtotal plus unknown fares**, never a complete trip price. Currencies are not combined or converted. Baggage and airport/station transfers may cost extra.

## Saving and consistency

**Use this mixed trip** replaces the previous transport plan atomically to avoid counting both replaced flights and new ground legs. The chosen trip, map, transport timeline, accommodation dates and budget use the selected journeys. Train and bus changes within a single provider itinerary remain grouped as one city-to-city journey; individual vehicle segments and operators are retained.

Unknown local times, overlapping legs and insufficient transfer windows block combination. The planner uses a conservative three-hour same-city interval when either adjoining journey is a flight, otherwise one hour; this is not a verified transfer time. Expired flight quotes remain visible as planning estimates, but must be refreshed before saving a new selection. Ground snapshots for different legs/dates/evening settings are saved separately, bounded to 24 snapshots; mismatched requirements and old schedules are excluded.

Mixed stay dates follow actual selected transport's local arrival/departure dates. Airport catalog anchors support hotel lookups after ground arrivals; an anchor is not represented as the ground boarding station or an airport you will depart from. No hotel night or accommodation charge is automatically removed because a journey crosses midnight.

## Validation

- 352 regression tests across 53 files passed, including individual fare preservation, all three transport modes, overnight and DST handling, unknown prices, entered fares, currency isolation, overlapping legs, multi-leg persistence, atomic replacement, partial budgets, and mixed accommodation dates.
- TypeScript and changed-file ESLint checks passed.
- Live browser check on Vancouver → Boston → Atlanta → Chicago → Vancouver, January 18–30, 2027: fresh independent flight quotes returned five complete combinations starting at USD 1,871 for three travelers, plus separate leg alternatives.
- Boston → Atlanta evening search returned a C&J/Peter Pan/Greyhound connection, departing January 22 and arriving January 24. Selecting it produced a USD 1,502 known flight subtotal plus an unknown bus fare. The displayed estimated daytime use changed from 20h 48m to about 19h 48m before the final boarding-time refinement; these are local-clock planning estimates, not verified savings.
- Saving that temporary mixed plan updated the selected map, chronological schedule, partial budget and Atlanta stay dates. The test selection was removed afterward, restoring the user's original unselected transport state.

No flights, tickets, stays, reservations or payments were booked. No paid Duffel calls or additional AI model calls were required for these comparisons.
