# Booking handoff and purchase safety

## Current application behavior

The planning flow and purchasing flow are separate. `prepareBookingHandoff` revalidates the selected offer through its provider adapter, checks that its expiry, live status, currency, total, and required baggage-inclusive price are usable, then validates the current seller identity, approval record, handoff method, and supplier URL through the central provider registry. It returns one of `blocked`, `approval_required`, or `ready`. Only `ready` includes a URL. A reprice that changes price, baggage, terms, seller, destination URL, or itinerary returns the updated offer with reasons and requires the user to approve the new snapshot before another handoff request.

Connection protection is surfaced in the handoff snapshot. A connection not explicitly marked provider-protected is described as a possible separate-ticket/self-transfer risk. Conditions and baggage allowance details remain attached to the snapshot; a missing required baggage total blocks the flow.

`submitApprovedPurchase` is a provider-neutral future transaction boundary. It requires explicit approval tied to the current revalidated snapshot, a current repriced offer, approved seller evidence, a configured purchase-capable adapter, and a unique idempotency key. It never accepts raw card data (only an opaque supplier/payment-provider reference) and never returns `confirmed` on exception or ambiguous supplier response. Deployments must implement `IdempotencyStore` durably and atomically (for example PostgreSQL unique keys); in-memory deduplication is insufficient across instances. Passenger data must be minimized by each future adapter and protected according to its retention policy.

## Provider capability status (this application)

| Provider / integration | Search | Revalidation | Link handoff | Authorized purchase workflow | Current app status |
|---|---|---|---|---|---|
| Duffel Flights | Documented; adapter exists, but no token or search enablement in this workspace | Duffel single-offer retrieval is implemented and mapped as repriced/validated, not booking confirmation | No itinerary-specific referral/official-airline URL is exposed by the current offer adapter; none is synthesized | Duffel Orders can create an airline booking and support instant payment or eligible holds; this application has no order adapter, permission confirmation, approved payment flow, final price/order reconciliation, or confirmation handling | Search/reprice code only; live access unverified; handoff and purchase not supported in this application |
| Tictactrip | Documented partner search; no configured live adapter/credentials | Unknown / not implemented | Search response has a referral reference, but seller/domain approval and an approved redirect have not been configured | Documentation describes booking/ticket issuance; application purchase adapter is absent and partner entitlement unresolved | No app handoff or purchase |
| Lufthansa Group Partner APIs | Partner access pending; no adapter | Unknown for configured product | Official-airline deep link is documented for partner plan; not configured | Partner NDC/order path requires separate commercial access; not integrated | No app handoff or purchase |
| Booking.com Demand API | Managed affiliate approval pending; no adapter | Search/price preview capabilities documented, not configured | Referral links documented, but not configured | Booking API capability exists for some flows; this app does not use it | No app handoff or purchase |
| Expedia Rapid Lodging | Partner approval pending; no adapter | Price check documented, not configured | Referral capability not established for this app | Booking API documented, not configured or entitled | No app handoff or purchase |
| Google Flights | No authorized structured API configured | No structured revalidation | Consumer comparison page only | None for this app | Never used as an offer or booking handoff |

Documentation of a provider endpoint is not evidence that this application has credentials, commercial permission, route coverage, or authorization to transact. The provider registry intentionally has no active seller approvals by default. No provider purchase is currently supported.

### Referral versus Duffel API purchase

Current Duffel documentation supports creating an order from exactly one selected offer. Orders usually take payment at creation (`type: instant`) or may be held only when the offer allows delayed payment (`type: hold`). Duffel documents passenger personal details and payment details as order inputs and warns that order creation initiates reservation/payment and may be non-refundable. This is a supplier API purchase, not a referral redirect. Its availability at the API/documentation level does not satisfy this app's missing account entitlement, seller authorization, passenger-consent, payment-provider, order-status/reconciliation, or duplicate-protection implementation requirements. We therefore do not enable this path here.

No airline-direct booking URL was found in the Duffel offer fields used by the current integration. An airline homepage or Google Flights comparison URL is not itinerary-specific and is not an acceptable substitute. An airline direct link can only be used if a separately authorized supplier integration returns a documented itinerary-bound deep link and seller evidence that passes the same registry policy.

## Required work before enabling a provider

1. Obtain and record provider account entitlement, commercial terms, production credentials, and authorized markets.
2. Capture the actual seller identity and approved supplier URL from the supplier response. Do not convert an offer owner or source-provider identity into the seller.
3. Add or renew an evidence-backed seller approval and test its domain and handoff method using the provider registry.
4. Implement adapter-specific offer revalidation, expiry and availability handling, and compare all material fields before presenting the approval step.
5. For purchase APIs, document passenger fields and data retention, validate the exact itinerary and final total, use supplier-supported payment/token flows, implement durable idempotency, handle pending/confirmed/failed supplier statuses, and reconcile ambiguous network outcomes before retries.
6. Revalidate again immediately before any transaction. Do not use a browser URL handoff as proof of booking.

Raw card numbers, security codes, and magnetic-stripe data must never enter application storage or logs. Only use a payment-provider or supplier token/reference where a separately approved integration supports it.

## Eligibility diagnosis (2026-10-18)

No configured provider currently qualifies for a live flight handoff. This is expected from several independent gates, not one missing toggle:

1. **Environment/configuration:** the workspace contains `.env.example` only. `DUFFEL_ACCESS_TOKEN` is absent and `DUFFEL_FLIGHT_SEARCH_ENABLED` is not enabled, so neither live search nor live repricing can be demonstrated here. The application does not have Duffel account entitlement or commercial terms recorded.
2. **Offer response semantics:** Duffel's authenticated offer has `owner` (the airline that provided the offer), but the mapper deliberately stores that in `providerOfferOwnerId` / `providerOfferOwnerName`. It leaves `sellerId`, `sellerName`, and `bookingUrl` null because the official response contract does not establish a seller-bound consumer booking URL. The operating carrier is captured separately from segment data. The provider offer owner must not be promoted to “actual seller” without evidence of the commercial seller relationship.
3. **Provider registry:** Duffel is only `documentation_reviewed`, not `verified`; it is a search/offer source and has no seller category. `providerRegistryConfig.approvals` is empty by design. Neither Duffel nor any airline/agency seller has an active evidence-backed approval record.
4. **Handoff boundary:** `prepareBookingHandoff` requires selected offer ID/provider to match the adapter, live/unexpired offer, revalidated applicable total and required baggage, exact seller identity, approved seller/method, and a supplier URL within the approval's exact domain scope. No Duffel handoff adapter or link is configured. The API reprice route only returns the normalized offer; it does not make it booking-ready.
5. **Other registered providers:** Tictactrip, Lufthansa Group, Booking.com, Expedia, and Viator have no working relevant adapter and/or lack partner credentials/approval. Google Flights remains comparison-only and is not a structured-offer or booking-link source.

The URL validator restricts a handoff URL and embedded redirect destinations to domains recorded in the seller's active approval. Merely listing a second domain as an official domain in the registry does not approve it for this seller role. Handoff now takes seller ID and display name from the revalidated normalized offer, rejects missing seller identity, and does not substitute Duffel's offer owner for the seller. The provider-specific mapping of carrier/seller identifiers still needs review when an authorized supplier and corresponding seller approvals are configured.

## API references reviewed

- Duffel [offer resource and single-offer retrieval](https://duffel.com/docs/api/v2/offers/price-offer): offer expiry is explicit; retrieving an offer provides complete, up-to-date information but does not guarantee it remains available at purchase time. Offer owner means the airline that provided the offer, not necessarily a verified seller.
- Duffel [orders API](https://duffel.com/docs/api/v2/orders): order creation starts a booking with the airline and supports payment at creation or eligible holds; it remains unimplemented in this app.
- Duffel [order and booking creation response handling](https://duffel.com/docs/api/overview/response-handling/order-and-booking-creation): transactional calls may take up to 120 seconds and must be handled without treating a timeout as proof of failure or success.
- Product-specific access requirements and research are recorded in [PROVIDER_RESEARCH.md](./PROVIDER_RESEARCH.md) and [DUFFEL_FLIGHTS.md](./DUFFEL_FLIGHTS.md).

Provider documentation was reviewed 2026-10-18 for this workflow. Recheck current docs, access and contract terms before enabling any transaction path.
