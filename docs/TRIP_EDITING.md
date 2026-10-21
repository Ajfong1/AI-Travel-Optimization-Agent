# Conversational trip editing

`POST /api/trip/edit` accepts a natural-language change and the current validated `TripRequirements`, then returns a Zod-validated, field-scoped result. The deterministic interpreter is separate from the conversation intake parser and only applies supported, unambiguous edits. Unsupported requests and ambiguities leave the input requirements unchanged. Mandatory-stop conflicts are reported without mutation.

## Supported edits

- Hostels instead of hotels, while retaining quality preferences.
- A numeric or spelled-out checked-bag count per existing traveler, retaining other bag counts.
- Flexible return airports while retaining departure-airport selection.
- Explicit overnight-travel permission.
- Cost-reduction and minimum-time-savings planning directives.
- Add a named country while retaining the existing transportation budget (asks for destination or a transport budget if either is missing).
- Remove a named, non-mandatory destination while retaining every other destination requirement.
- Restrict flight recommendations to verified official-airline sellers; the route optimizer continues to permit official rail/bus operators.
- Explicit ISO departure/return date changes, with derived trip-night count updated when both dates are known.

`TripRequirements.planningDirectives` retains relative savings/time goals that do not fit a hard budget or the existing optimization-mode enum. These are marked user-provided and are not represented as guaranteed savings or hard caps. A comparable selected-route baseline is required to prove a relative target.

When a budget explicitly includes the `transport` category, the route optimizer enforces it as a hard ceiling using integer minor units. Routes over the ceiling are excluded. If the route currency differs from the budget currency, budget compliance is unverified and the route cannot enter the verified-cheapest set. A full-trip budget is not treated as a transportation-only ceiling.

## Change impact

Results report changed fields, datasets to invalidate, whether route optimization should run again, and whether previous candidate routes remain comparable or must be held as stale until affected data is refreshed. Airport/date/destination/group/mode changes invalidate relevant transport searches; baggage changes invalidate baggage pricing and route totals; accommodation preference/date/group changes invalidate accommodation offers; route/date/destination changes invalidate the activity schedule. Preference-only ranking changes can reuse unexpired supplier offers.

This phase provides the impact plan to the caller; it does not execute provider re-searches. The application has no persistent selected-route repository or orchestration endpoint to coordinate a refreshed graph yet. Previous candidate routes must not be labeled current-priced after the result says `retain_as_stale_until_refetched`.

The dashboard editor sends the same current requirements to this endpoint, updates its browser-session draft only for applied results, and displays clarification/conflict messages otherwise. Session storage is not durable trip persistence. Purchasing remains outside this flow.
