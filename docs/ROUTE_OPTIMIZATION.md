# Transport route optimization

`src/server/optimization/route-optimizer.ts` implements deterministic route generation over a finite graph of supplier offers. It does not query suppliers, invent edges, or call a language model. Candidate destination generation and provider discovery remain upstream responsibilities.

## Input contract

- Pass schema-validated `TripRequirements` and date windows. Date-specific search stops with a blocker when either trip boundary is absent or needs clarification.
- Pass place nodes and explicitly map acceptable origin and return airports/stations to `originPlaceIds` and `finalPlaceIds`. A different return node supports open-jaw trips.
- Pass one directed graph edge per real offer/route between place nodes. Include actual transport legs, schedules, source metadata, and the result from `priceTransport` as `baggageCost`.
- Set `sellerApproval: "approved"` only after the provider registry has approved the actual seller and booking destination. Unverified and revoked sellers are excluded.
- Pass provider search coverage records, including unavailable or unresolved pairs, so the result can report coverage gaps.

The edge baggage cost must use one common currency across all legs of a candidate route. Missing cost, incompatible currency, incomplete baggage evidence, stale offers, missing seller identity, or unconfirmed availability keeps the itinerary out of `verifiedCheapestRoutes`. A search result is not upgraded to supplier booking readiness by this engine; offer validation statuses remain attached to each result leg.

## Search and ranking

The engine searches a bounded beam (`beamWidth`, `maxStops`) over chronologically compatible, directed offer edges. It respects exact mandatory destinations, exclusions, trip date windows, optional stop uniqueness, configured minimum stays, trip duration, overnight allowance, and the existing connection-feasibility screen. Supplied schedules are evaluated in their instants; timezone fields are retained in source legs and used by `validateTransportRoute` for overnight detection.

- `lowest_cost` orders by complete baggage-inclusive group transportation cost and does not penalize duration.
- `balanced` orders by cost plus caller-configured travel-hour, connection, overnight, and inconvenience penalties, all expressed in the common minor-unit currency.
- `more_places` uses deterministic explicit destination preference and interest-tag matches, number of stops, then cost. It does not accept model-generated scores.

Results preserve destination order, offer/leg source, seller, timestamps, price, duration, available distance, connection risks, warnings, and assumptions. The response always says the search is bounded and does not claim global optimality. Only the bounded final itinerary selection is returned; unpriced routes may be shown with a warning, but never enter the verified-cheapest list.

## Coverage and limitations

This module cannot prove that an upstream provider search is exhaustive or that offers remain bookable. It reports missing coverage metadata, excluded sellers, and no-route outcomes. Repricing/revalidation is not initiated here because adapters own their supplier-specific validation operation; call an approved adapter's validation/repricing method for the leading offer IDs, rebuild affected baggage-inclusive edge costs, and rerun optimization before handoff. Booking readiness still requires the supplier's documented confirmation and a separate explicit user approval flow.

The bounded beam is a heuristic. It can omit a globally cheapest itinerary when the search space exceeds its width. It does not currently enumerate flexible stay schedules beyond those represented by the supplied offer departure times, or infer station/airport transfer durations. Those values must come from upstream data. Accommodation and activity economics are deliberately out of scope until the initial transport route is selected.
