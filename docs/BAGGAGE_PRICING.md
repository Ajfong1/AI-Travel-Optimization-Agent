# Baggage-aware transport pricing

`src/server/optimization/baggage-pricing.ts` is the provider-neutral boundary for calculating comparable transport costs. Call `priceTransport` with a normalized fare/ticket grouping, per-traveler bag requirements, and evidence-backed rules for every segment. Feed only `eligibleForVerifiedCheapest` totals to a verified-cheapest ranking; `comparableBaggageInclusiveTotal` is the optimizer-facing helper.

## Cost semantics

- `ticketGroups[].fare` is the provider's authoritative fare amount for that ticket group and all listed passengers. Add a ticket group once even if it has multiple segments. When separately booked legs exist, model separate groups so each ticket fare and rule is counted independently.
- `baseFare` and `taxes` are display/audit components of the authoritative fare; they are not added again. `mandatoryCharges` is for explicitly required charges that are additional to `fare` (for example, a required seat if no compliant seat is available without it). Do not copy fare-included taxes or charges into that list.
- Bag allowance rows are scoped to ticket group, segment, traveler, and bag kind. A positive allowance must be explicit: absent data is unknown, not zero or included. Fee rows are explicit per-item amounts and must be evidenced by the supplier or verified provider data.
- The engine adds only the shortfall beyond an included allowance. It never carries allowances between segments or separately issued tickets. Mark later self-transfer segments `selfTransferRecheck` so missing allowance data produces an explicit recheck uncertainty.
- Rules must describe accepted weight and dimensions whenever the traveler supplies size/weight constraints. A missing or smaller limit makes the candidate unverified.
- `fare` includes the passenger count represented by `passengerIds`; every passenger's baggage is calculated individually. Per-person output currently reports baggage totals because provider fare amounts may be group-priced and are not safely divisible where passenger fares differ.
- Currency conversion requires an explicit directed rational rate. Conversion uses integer arithmetic and half-up rounding to the target currency's minor-unit exponent. Rate source and retrieval time are retained in input; stale-rate policy is a caller/provider concern and should be applied before ranking.

Rail and bus use the same segment `mode`, allowance, fee, size, and evidence model. A mode with no material bag fee can still be verified only when its applicable baggage rules are explicitly supplied.

## Provider integration boundary

The Duffel mapper currently preserves segment/passenger bag counts from its offer response, but this is not sufficient to populate all requirements here: required item dimensions/weights and explicit per-bag fee scope/prices must be sourced and validated. Do not translate a missing Duffel allowance or ambiguous ancillary-service amount into a zero fee. Until an authorized supplier response supplies the missing details, the candidate must remain out of verified-cheapest ranking. The normalized offer's legacy flattened `mandatoryBaggageAmountMinor` is intentionally not accepted by the optimizer comparator; use evidence-bearing `TransportPricingInput` instead.

Provider search/repricing results are snapshots, not purchase authorization. Rebuild the pricing input from the latest validated offer before a booking handoff, keep its source timestamps, and do not treat the calculator as a booking-readiness check.
