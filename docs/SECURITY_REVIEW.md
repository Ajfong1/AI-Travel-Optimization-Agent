> **Historical snapshot:** This records an earlier engineering stage. Use [Current status](CURRENT_STATUS.md) and [Local setup](LOCAL_SETUP.md) for current features and setup. Earlier inventory, storage and test-count statements below are historical.

# Security and privacy review

**Review date:** 2026-10-18  
**Status:** no purchase workflow or production user accounts are enabled. The application is **not production-ready** because public API protection, persistent-data controls, production provider authorization and deployment controls are not in place.

## Scope and positive controls

- Duffel and Tictactrip credentials are read from server-side environment variables; there is no `NEXT_PUBLIC_*` provider secret.
- `.env.local` is ignored by Git. The audit confirmed the ignore rule and scanned tracked/source files for common Duffel, Stripe and AWS key patterns without emitting secret values; no matching pattern was found in scanned source or the initial commit.
- Duffel API origin is fixed in the server client, redirects are disallowed, calls have timeouts and bounded retries, and API errors are normalized rather than logging provider response bodies.
- Signed HMAC state binds pagination/revalidation tokens to an offer, baggage request, provider mode and expiry. A token becomes invalid when the Duffel credential changes.
- Provider URL checks require HTTPS and exact approved seller domains; reject URL credentials, unsafe/IP/localized hosts, look-alike hosts, opaque redirect destinations and redirects outside approval scope. Registry approval is role- and date-specific; a domain alone does not establish seller trust.
- Provider-test offers are explicitly labeled and blocked in production. Unknown provider mode is withheld. The optimizer requires live mode and valid current price/baggage/seller data for verified-cheapest status.
- Booking service requires explicit approval, a matching post-revalidation change key, actual seller evidence and an idempotency claim before calling a purchase-capable adapter. No purchase adapter or purchase API route is currently configured. Unit tests use fake adapters and do not spend money.
- Passenger/payment data is not logged by the booking service. The application does not store raw card details.
- Intake/edit request bodies are Zod-validated; provider search bodies have strict schemas. There are no provider or personal-data log statements in the application source inspected.

## Findings and residual risks

### High: no user authentication or authorization

All current application/API routes are unauthenticated. Intake and trip edit are stateless processing endpoints, but the browser dashboard stores a draft in `sessionStorage`; there is no user identity, account ownership check, saved-trip persistence or authorization model. Do not store sensitive passenger/traveler data or expose this as a multi-user production service until authentication, authorization, session protection and per-user data access controls are implemented.

### High: rate limiting is process-local and deployment-dependent

The application has a process-local 20-request/minute guard for provider API routes; it is neither distributed across instances nor an adequate abuse-control boundary by itself. The audit also found that trusting caller-provided `x-real-ip` by default permits rate-limit bucket spoofing. The code now ignores that header unless `TRUST_PROXY_REAL_IP_HEADER=true` is explicitly set. Keep it false unless the edge proxy strips/replaces the incoming value. Prefer shared/edge rate limiting for all public routes, including intake/edit. Without a trusted client IP, the fallback bucket is shared and can throttle unrelated users together. Add request quotas and monitoring before public launch.

### High: live supplier identity and booking readiness unavailable

The real Duffel test-mode search and refresh succeeded, but the returned test offers did not identify the actual booking seller or provide an itinerary booking URL. Duffel owner/operating-carrier metadata is not treated as the seller. Seller validation therefore blocks handoff as intended. Production API access, commercial permission, live offers, seller evidence and an authorized handoff path remain unverified. No seller is automatically approved.

### Medium: test/live credentials are configuration-sensitive

`DUFFEL_ALLOW_TEST_MODE` is refused in production and test-mode offers carry `liveMode: false`. Keep test credentials and test-mode enablement out of deployment secrets. Production must use a live authorized token, `DUFFEL_ALLOW_TEST_MODE=false`, and the live response mode check. Health configuration status is presence-only, not an authentication test. Do not infer successful provider access from it.

### Medium: development dependency advisory remains

`pnpm audit --prod` returned no production advisories. Full `pnpm audit` reports one high-severity development chain: `braces <=3.0.3` via `eslint-config-next` → `@next/eslint-plugin-next` → `fast-glob` → `micromatch`. The package registry reported no patched release at audit time; this is in the lint/development chain, not the production dependency tree. Track and re-audit before release. Drizzle ORM and the vulnerable dev esbuild version were upgraded/overridden during this audit.

### Medium: database schema and privacy lifecycle are not operationally validated

The dashboard currently uses session storage and no DB repository/migration is exercised. The schema includes required seller columns while providers may legitimately return unknown sellers; review nullable seller evidence before persisting such offers. Define retention, deletion, encryption, access logging and backup/restore policies before storing traveler or itinerary data.

### Low/medium: UI and browser security checks are incomplete

No automated browser, accessibility, CSP/security-header, CSRF, visual-regression or responsive test setup exists. The only server changes are currently stateless search/intake/edit flows, and no purchase action is exposed. Before adding mutations/accounts, implement origin/CSRF policy, security headers/CSP, output encoding review and authorization tests. Verify external-link `rel` behavior once booking links are rendered.

### Provider policy/licensing obligations unresolved

Duffel, Tictactrip and Booking.com credentials/configuration do not by themselves establish API access or commercial/referral approval. Ground inventory lacks sufficiently verified baggage fees and seller identity. Activity/accommodation providers require documented access, attribution/licensing and data-retention review before use. Google Flights has no structured fare-data integration in this code; it is not scraped or automated. Recheck official provider terms and applicable privacy/legal requirements with the organization before commercial deployment.

## Before a production launch

- Implement authentication/authorization and a user-owned persistent trip model.
- Move public API rate limiting to a shared trusted edge/service; monitor abuse and supplier quota.
- Rotate any credential previously pasted into chat; configure a fresh server-only token. Never include its value in support logs, screenshots or commits.
- Complete live Duffel permissions and test safe live search/reprice with actual seller and baggage evidence; preserve purchase disabled until separately approved.
- Run production dependency and secret scanning in CI, including commit history; enable dependency updates and advisory alerts.
- Set TLS, secure cookies/session policy, security headers/CSP, database encryption/backups, least-privilege DB credentials, retention/deletion procedures and incident response.
- Add structured redacted observability, error/latency alerts, shared rate-limit metrics, provider-availability monitoring and a tested restore procedure.
