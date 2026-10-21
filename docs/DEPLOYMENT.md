> **Historical snapshot:** This records an earlier engineering stage. Use [Current status](CURRENT_STATUS.md) and [Local setup](LOCAL_SETUP.md) for current features and setup. Earlier inventory, storage and test-count statements below are historical.

# Deployment and operations

**Current status: development only.** A successful local build does not make this application production-ready. The authenticated Duffel call verified earlier was test-mode inventory; there is no demonstrated production booking or purchase path, no user authentication, no live ground/lodging/activity inventory, and no persistent trip repository.

## Runtime and local development

Requirements in `README.md`: Node.js 20.9+, pnpm 9+, and PostgreSQL 15+ for database migrations. From the repository root:

```powershell
pnpm install
Copy-Item .env.example .env.local
pnpm dev
```

Open `http://localhost:3000`. Keep the development server running in its terminal. The UI supports natural-language intake and browser-session draft editing; transport search is not wired into the dashboard. The Duffel API can be called server-side when authorized configuration is present. Test offers must remain visibly identified as test data and are not bookable flights.

For a local Windows terminal where `pnpm` is not on `PATH`, install/activate Node and pnpm through the normal developer setup, or use the environment-provided pnpm path shown by the local Codex runtime. Do not make a machine-specific cached runtime path a project requirement.

## Environment variables

Copy `.env.example` to `.env.local` for local Next.js use. Keep `.env.local` out of version control. Production secrets should be set using the deployment platform's server-side secret manager, never compiled into browser variables.

| Variable | Purpose | Required state / caution |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection for Drizzle migration/runtime configuration | No application persistence layer has been exercised; use least-privilege credentials and a managed/private DB endpoint. |
| `DUFFEL_ACCESS_TOKEN` | Server-side Duffel API token | Test token only for test-mode; production requires authorized live access and permitted markets. Never use `NEXT_PUBLIC_`. |
| `DUFFEL_FLIGHT_SEARCH_ENABLED` | Enables Duffel search/reprice adapter | Keep `false` until token/access confirmed. |
| `DUFFEL_ALLOW_TEST_MODE` | Allows Duffel test data in non-production | Set `false` in production. The backend also rejects provider test mode under `NODE_ENV=production`. |
| `TICTACTRIP_API_TOKEN` | Server-side partner API token | Partner authorization required. |
| `TICTACTRIP_PRICE_CURRENCY` | Currency required by the ground fare adapter | Confirm interpretation with provider before ranking. |
| `TICTACTRIP_GROUND_SEARCH_ENABLED` | Enables Tictactrip adapter | Keep `false` until access, coverage, seller, baggage and terms are verified. |
| `TRUST_PROXY_REAL_IP_HEADER` | Allows rate limiter to read `x-real-ip` | Leave `false` unless trusted ingress overwrites this header on every request. Prefer shared edge limiting regardless. |
| `BOOKING_COM_*` | Reserved for a future Booking.com Demand API integration | Not implemented/enabled. Managed Affiliate Partner/API approval is not implied by the variables. |
| Generic `*_PROVIDER_API_KEY` / `*_BASE_URL` placeholders | Future provider configuration placeholders | Not wired provider integrations; setting these does not enable search. |

The `/api/health` route reports configuration presence only. `configured_unverified` does **not** mean the provider token authenticated, an account is authorized, inventory exists, or the DB is reachable.

## Database setup and migrations

Drizzle schema and SQL migrations are under `src/server/db/schema` and `drizzle/`. Before using a database:

1. Provision PostgreSQL 15+ in a private network, create a dedicated least-privilege application/migration role, and set `DATABASE_URL` in the command environment or platform secret store.
2. Review pending SQL with `pnpm db:generate` where appropriate; review generated SQL before applying it.
3. Apply migrations with `pnpm db:migrate` against a disposable database first.
4. Verify rollback/restore and migration behavior on staging.

Database migrations, connection pooling, persistence queries, backups and restore have not been tested in this audit. The dashboard keeps draft requirements in `sessionStorage`; there are no saved user trips. The current transport offer schema also requires seller ID/name although the Duffel adapter correctly represents unknown seller identity as `null`; reconcile this schema before adding offer persistence.

## Production build/run (not yet approved for launch)

For a controlled staging deployment after blockers are resolved:

```powershell
pnpm install --frozen-lockfile
pnpm typecheck
pnpm lint
pnpm test
pnpm build
pnpm start
```

Use a supported Next.js hosting environment with a stable Node runtime, HTTPS, secure environment-variable injection, a trusted proxy/edge rate limiter, and a private managed PostgreSQL service. The repository has no Dockerfile, CI deployment pipeline, infrastructure-as-code, environment-specific migration job, or rollback automation. Define these for the target platform rather than assuming a hosting setup.

## Backups and operational monitoring

No application backup/monitoring integration currently exists. Before storing data, configure:

- Automated encrypted PostgreSQL backups and point-in-time recovery according to an approved retention policy; run restore drills and verify RPO/RTO.
- Redacted structured logs with request correlation, provider ID, safe error code, response latency, inventory counts and validation status. Never log credentials, passenger PII, full provider responses, signed state tokens or payment data.
- Alerts for provider authentication/permission failures, rate limits, timeouts, malformed response/schema changes, empty/unavailable inventory, unexpected test-mode responses in production, DB connection errors and elevated 5xx/latency.
- Shared rate limiting and provider quota tracking at ingress; the in-process limiter is only a local guardrail and is not reliable across replicas.
- Secret rotation, least-privilege credentials, dependency/advisory scanning and incident response.

## Launch blockers

- Authorized Duffel live-mode credential/account/market permissions and a completed live search/reprice validation; current API evidence is test mode only.
- Actual seller and approved supplier/referral link evidence for any booking handoff; current Duffel search responses lacked seller/link data.
- Provider authorization and sufficiently complete pricing, baggage, availability, licensing/attribution and terms for rail, bus, accommodation and activities.
- Persistent trip database repository, auth/authorization, retention/deletion policy, shared rate limiting, production observability and tested backup/restore.
- Browser E2E/accessibility/responsive/security testing and deployment-specific ingress/header review.
- Resolve or explicitly risk-accept the development-only high `braces` advisory; re-run the full dependency audit at release time.

Until these are complete, deploy only to a development or isolated internal test environment; do not accept customer bookings or present test fares as live bookable offers.
