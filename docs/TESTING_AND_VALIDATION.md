> **Historical snapshot:** This records an earlier engineering stage. Use [Current status](CURRENT_STATUS.md) and [Local setup](LOCAL_SETUP.md) for current features and setup. Earlier inventory, storage and test-count statements below are historical.

# Engineering audit: testing and validation

**Audit date:** 2026-10-18  
**Disposition:** useful development foundation; **not production-ready**. The checks below distinguish application tests from supplier test-mode validation and from unperformed production/browser/database tests.

## Verification performed

| Area | Result | Evidence and limits |
|---|---|---|
| Duffel supplier connectivity | **Pass, test mode only** | A live HTTP request through `/api/flights/search` authenticated with the configured server credential and returned 44 Duffel offers for SFO–LHR, one adult, economy, one checked bag and one carry-on. Response mode was `provider_test`. |
| Duffel offer refresh | **Pass, test mode only** | A selected offer was refreshed through the app's signed validation endpoint. Duffel returned `repriced_or_validated`; the USD total and required baggage status were preserved. This is not booking confirmation. |
| Live commercial flight inventory | **Not verified** | The credential is test-mode. Duffel test schedules/prices are not production inventory. The sample response did not identify an actual seller or return an itinerary booking URL. No orders or purchases were attempted. |
| Rail/bus offers | **Not verified** | The Tictactrip adapter is partner-gated and disabled unless the relevant authorization and configuration are present. Current normalized results lack complete per-item baggage and approved seller handoff evidence. |
| Accommodation/activities | **Not live-integrated** | Planning, validation and ranking modules exist, but no configured authorized live provider adapter/inventory was demonstrated. Dashboard values remain empty states. |
| Currency and baggage arithmetic | **Unit-tested** | Integer minor-unit arithmetic, zero/two/three/four-decimal currency conversion, rounding, group/per-person bag pricing, separate tickets, incomplete baggage data and repriced fares are covered. Budget parsing now uses the currency exponent and fails closed on unrepresentable precision. |
| Route optimization | **Unit-tested** | Fixed and flexible windows, alternate endpoint airports, open-ended and mandatory destinations, mixed modes, missing baggage totals, expired offers, test-mode exclusion, budget rules, cross-offer connection counts/risks, and a small independent exhaustive enumeration are covered. The exact enumeration is a bounded synthetic case, not a proof for production-scale graphs. |
| Provider safety and handoff | **Unit-tested, no live handoff** | Tests cover approval/domain policy, unknown/look-alike sellers, redirects, expiration, changed price, unavailable inventory, missing seller, consent, idempotency and supplier failures. Duffel's observed offer lacks the seller/link evidence needed for handoff. |
| Accommodation/activity planning | **Unit-tested with supplied data** | Deterministic pricing, ranking, itinerary/opening-hour rules and full-trip comparisons have module tests. These tests do not prove provider licensing, live availability or commercial access. |
| Frontend | **Static/code review only** | Intake and dashboard empty/loading/error states are implemented. The dashboard explicitly says transport search has not run. No Playwright/Cypress/browser automation or accessibility scanner is configured; responsive behavior, keyboard/screen-reader behavior, link destinations and visual breakpoints were not formally exercised. The map-like hero artwork is illustrative, not a live route map. |
| PostgreSQL | **Not exercised** | Drizzle schema and migrations exist, but no database connection, migration run, persistence integration or restore test was demonstrated. Trips are held in browser `sessionStorage`, not saved to PostgreSQL. |
| E2E/performance | **Not configured/measured** | No E2E script/framework or load/performance baseline is present. |

## Automated commands

After the audit fixes and dependency updates:

- `pnpm test` — **110 tests passed** across the Vitest suite.
- `pnpm typecheck` — passed (`tsc --noEmit`).
- `pnpm lint` — passed (ESLint).
- `pnpm build` — passed (Next.js production build).
- `pnpm audit --prod` — no production dependency advisories reported.
- `pnpm audit` — one high-severity **development-only** transitive advisory remains: `braces <=3.0.3` through `eslint-config-next` → `@next/eslint-plugin-next` → `fast-glob` → `micromatch`. The registry reported no patched version when checked. No production dependency path was reported for this finding. Recheck on dependency updates.

The audit initially found Drizzle ORM below the patched version for [GHSA-gpj5-g38v-94v9](https://github.com/advisories/GHSA-gpj5-g38v-94v9); it was upgraded to `drizzle-orm ^0.45.2`. The `drizzle-kit` development dependency was updated to `^0.31.11`, and its vulnerable transitive esbuild was overridden to `^0.25.0`, resolving the moderate esbuild advisory. `pnpm audit` still reports the braces finding above.

## Reproduced findings and changes

1. **Expired offers could remain in the optimizer's current-price pool.** Route freshness checked retrieval age and supplier availability, but not an offer's explicit expiration. The optimizer now requires a valid expiry later than the evaluation time and emits an expiry/freshness warning otherwise. Regression test: `excludes supplier offers whose explicit expiration has passed`.
2. **Currency budget parsing assumed two minor-unit digits.** A JPY or KWD budget could be incorrectly compared. Budget strings now use the same currency exponent rules as cost conversion; excessive precision or unsafe values make budget compliance unverified and exclude the route from verified-cheapest results. Tests cover JPY, KWD and CLF.
3. **Client-controlled `x-real-ip` could split rate-limit buckets.** The rate limiter now ignores that header by default. Trusting it requires `TRUST_PROXY_REAL_IP_HEADER=true`; only set this when a trusted reverse proxy strips/replaces the incoming header. Regression tests cover both modes.
4. **Health response was stale and misleading.** `/api/health` said no supplier integration was connected even when configured. It now reports only whether configuration is present and explicitly says connectivity/access are not verified; it never returns credential values.
5. **Inter-offer transfers were omitted from route connection counts.** Routes now include connections across supplier-offer boundaries and do not inherit the next offer's internal connection-protection claim across a separate ticket. Regression tests assert independent-ticket counts and unprotected warnings.

## Remaining validation work

- Obtain Duffel production/live credentials and explicit account/market authorization; verify live mode, live offer refresh, real seller metadata, baggage fee coverage, expiration behavior and approved handoff. Do not purchase during validation.
- Obtain and verify commercial/API access for ground transport, accommodation and activity providers; verify each product's licensing, current inventory, fees, attribution and authorized referral/booking process.
- Add a persistent database-backed trip repository and run migrations against a disposable PostgreSQL instance, including backup/restore validation.
- Add browser E2E tests for intake, partial clarification, dashboard edits, loading/empty/error states and narrow-screen layout; add accessibility checks and keyboard/screen-reader review.
- Add generated/property-based route cases and broaden exhaustive-vs-engine comparison across small graphs with time windows, transfer constraints, stop constraints, and alternate endpoints. Search is beam-bounded and never claims global optimality.
- Wire application-level balanced-mode weights and coverage orchestration. The optimizer's balancing penalties are configurable at its internal input boundary, but default to zero and are not yet exposed through a user setting/API; its coverage records are supplied externally and do not yet establish mode-by-mode/provider-wide completeness.
- Run deployment-environment security, rate-limit, observability, latency and load tests behind the actual trusted edge/proxy.
