# Xotelo long-stay pricing repair

The hosted Manila trip requested October 31–December 5, 2026 (35 nights). Xotelo returned HTTP 200 with an error envelope: `chk_out (2026-12-05) must be less than 1 months from chk_in (2026-10-31)`. Both a hotel and a hostel returned dated rates for a three-night request, then rejected the 35-night request. This was a provider date-range constraint, not missing Manila accommodation. The old adapter hid the underlying error and repeated the unsupported range for every sampled property.

The adapter now splits stays of 28 nights or more into contiguous periods of at most 21 nights, keeping every request below one calendar month even in February. Stays of up to 27 nights keep their original single request. The same property, guest counts, room count and currency are used for every period.

A full-stay quote is emitted only when all periods return fresh, matching dated rates. Its total is the sum of each period's nightly rate multiplied by that period's nights and requested rooms. Missing periods are never replaced with a sample rate, zero cost or extrapolated total. The earliest quote timestamp controls freshness. Candidate limits bound each city search to at most 20 rate checks, in addition to cached listing discovery; no paid fallback is used.

Long-stay results show the separately quoted periods, their dates, suppliers and totals. The same breakdown remains visible after adding a stay to the plan. The UI explains that room changes may be required and that a continuous room, bed allocation, taxes and availability need confirmation. These are planning estimates, not bookings.

## Verification

- Actual free-provider check for Manila, October 31–December 5, 2026, one adult, one room, USD, both requested property types: complete hotel and hostel quotes returned in 29.16 seconds.
- Unit tests verify a 35-night quote with different rates in each period, exact dates, room-count multiplication, no total when a period is missing, and calendar/leap-year boundaries.
- Lodging provider, saved-plan, room detail and one-way stay regressions: 51 tests passed. TypeScript and ESLint passed for changed source.
- No reservations, payments, sign-in emails or paid travel-provider requests were made.

The initial three-night/35-night diagnostic and the complete-stay benchmark use public Xotelo endpoints without credentials. Request and result diagnostics are kept in ignored local work files. Reference: [Xotelo rates documentation](https://xotelo.com/).
