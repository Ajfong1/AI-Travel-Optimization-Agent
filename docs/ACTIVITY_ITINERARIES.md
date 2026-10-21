# Activity discovery and daily itineraries

## Current access status

The provider research at `docs/PROVIDER_RESEARCH.md` identifies Booking.com Attractions as a selected-partner beta and Viator Full as requiring approval/certification for live prices and availability. Those access gates are not established in the provider registry. The activity module therefore defines an adapter contract but ships no live activity provider and sends no supplier requests by default. Suggestions supplied by a future licensed source must retain its source timestamp, license, and attribution. Do not substitute consumer-site scraping.

## Module boundaries

- `src/server/activities/types.ts` defines validated activity, source, pricing, hours, booking, preference, and itinerary contracts.
- `discovery.ts` runs injected adapters only when the central registry marks activity search verified. It validates provider response shapes and applies seller URL policy to live-bookable products. Preference matching is deterministic; `must_see`, `preferred`, `optional`, and `excluded` are represented independently from trip intake.
- `itinerary.ts` derives local-date stay windows from a selected route and matching accommodation plan, then schedules activities against local opening hours, event slots, durations, travel inputs, and a configurable daily limit.

## Integrity behavior

An activity marked live-bookable must provide a seller identity/link that passes the provider trust registry. Listings with unknown opening hours are scheduled only tentatively with an explicit warning; days published as closed and dates in a sourced closure calendar are never scheduled. Provider-sourced travel times are preferred. If absent, the scheduler uses a configurable transfer allowance and labels it as an assumption; it does not claim that allowance is a measured route time. Distance clustering is used only as an ordering heuristic.

Live verified prices, estimates, verified free activities, and unknown prices remain distinct. A paid live price contributes to verified totals only when the offer is available, seller validated, fee inclusion confirmed, and currency matches the requested summary currency. Estimates remain separate. Currency conversion and product-specific ticket/participant rules are not inferred; callers must supply explicit participant count for per-person/per-ticket totals.

## Future provider configuration

Before implementing a provider adapter, obtain the access level and terms described in `docs/PROVIDER_RESEARCH.md`, update the registry with reviewed capabilities and restrictions, and document licensing, attribution, quotas, supported geography, hours, event slots, pricing semantics, and referral behavior from the provider's official API documentation. Keep API credentials server-side. Revalidate timed inventory before handing a traveler to a supplier; this module does not purchase tickets.
