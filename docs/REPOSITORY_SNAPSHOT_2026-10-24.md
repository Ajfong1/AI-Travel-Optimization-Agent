# Repository snapshot — October 24, 2026

The application is saved in the private [wayfarer-hosting repository](https://github.com/Ajfong1/wayfarer-hosting). The public [AI-Travel-Optimization-Agent repository](https://github.com/Ajfong1/AI-Travel-Optimization-Agent) publishes its README and documentation. Repository visibility remains unchanged.

## Included in the source repository

- Complete `src/` application, domain modules, provider adapters and regression tests.
- Root package manifest, pnpm lockfile, TypeScript/Next.js/lint/test configuration and Next.js agent instructions.
- Dockerfile, Render configuration, deployment scripts and Supabase workspace schema/access policies.
- Safe `.env.example` placeholders, setup guides, user guide and dated bug-fix/benchmark reports.
- Python flight worker, compatibility tests, benchmark and diagnostic scripts, and the secret-checked source preparation script.

This includes password accounts and workspace sync, the dashboard refresh, major-airport gateways, multicity and one-way flights, Asian destination mappings, hotel/hostel discovery, long-stay pricing periods, ground schedules and mixed transport planning. See [Current implementation](CURRENT_STATUS.md) for behavior and limitations.

## Excluded

Real environment files and keys, saved user trips, account data, screenshots, raw live-search responses, quota ledgers, caches, Python environments, installed dependencies and build output remain outside the repository snapshot. The entire local `work/` directory is ignored. Source references to those files identify local development evidence, not missing application modules.

`scripts/prepare-hosted-source.mjs` builds an explicit source allowlist and checks it for configured credentials and recognized token patterns before producing an ignored upload artifact. GitHub file hashes are compared with the prepared source to verify the saved snapshot. The public repository receives documentation only.

## Reproducing checks

Use the [local setup guide](LOCAL_SETUP.md), then the development commands in the [README](../README.md). Python compatibility unit tests are in `scripts/test_fast_flights_compat.py`. Flight benchmarks and diagnostics under `scripts/` are opt-in and make public search requests; they are not part of the normal test suite. Historical efficiency reports require raw inputs that intentionally remain local.

This repository save does not rerun travel searches, send verification emails or deploy a new server version. The latest functional revision was already deployed and verified separately; see the [Asia report](ASIA_SEARCH_FIX_2026-10-24.md) and [long-stay report](XOTELO_LONG_STAYS_2026-10-24.md).
