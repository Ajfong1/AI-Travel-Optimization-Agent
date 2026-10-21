# Alternative-route ranking

`src/server/optimization/alternative-ranking.ts` ranks `OptimizedRoute` candidates after route generation. It performs no provider calls and accepts no model-generated prices or scores. Price eligibility, schedules, explicit provider classification, and convenience/experience metrics must come from the optimizer and approved provider registry.

## Configuration and use

Call `rankRouteAlternatives` with the candidate routes, selected optimization mode, search-coverage state, and a policy. `maximumPremiumBasisPoints` is required and configurable per user or application policy (for example, `1500` means a 15% cap); the ranking engine has no universal baked-in premium. Value weights are also required configuration. Monetary comparisons use integer minor units and `BigInt` for premium threshold and percentage calculations.

`rankOptimizationAlternatives` adapts a `RouteOptimizationResult`. For a wider alternative pool, configure the route optimizer's result limit to return a wider shortlist before ranking. Search completeness is derived from its coverage gaps and warnings.

## Categories

- **Cheapest verified**: least-cost current, baggage-inclusive, comparable-currency route from the supplied feasible candidates.
- **Best value**: a distinct Pareto-efficient route within the premium cap whose configured deterministic benefit score exceeds its price premium.
- **Fastest practical**: least elapsed supplier-journey time within the same premium cap, with connections, risk, and cost as tie-breakers. The cheapest route may fill this category if it is already fastest.
- **Preferred booking**: highest verified provider quality within the cap; provider type must be derived from seller/registry evidence. The cheapest route may fill this category when it already uses official or approved providers.

The best-value explanations name measurable deltas such as time saved, connections or self-transfers removed, provider verification improved, or cities/countries added. Each category contains the same route shape, currency, absolute price difference, increase in basis points and percent, coverage qualification, and explanation list for frontend comparison. Empty categories remain empty when no candidate qualifies.

The Pareto frontier treats lower price, vehicle time, elapsed journey time, connections, overnight journeys, independent tickets, and self-transfers as better; it treats higher provider quality, city/country count, convenience, and deterministic experience match as better. Exact metric equivalents for the same destination sequence are deduplicated. A candidate with missing/stale baggage-inclusive cost, mixed unconverted currency, incomplete schedule feasibility, or unknown provider verification is excluded with reasons.

The selected route remains controlled by the user's mode: cheapest selects the cheapest verified candidate, while balanced and experience select by their existing deterministic route score from the eligible Pareto recommendations. Incomplete search coverage labels all categories and prevents a global cheapest/fastest claim. A recommendation is not a booking confirmation; offers still require provider revalidation before handoff.

## Metric interpretation

`durationMinutes` is time in vehicles/aircraft. `elapsedJourneyMinutes` spans the first scheduled departure through final arrival, including waits and time at intermediate destinations; `tripElapsedMinutes` currently reports the same span. Connections and independent-ticket counts include transfers between separately supplied offers. When ticket-group metadata is missing, ticket relationship is unknown and must not be presented as protected. Self-transfer counts still depend on explicit upstream metadata.
