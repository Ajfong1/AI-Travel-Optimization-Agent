# Local setup

These commands require the complete application source. The initial GitHub publication contains documentation; it is not yet a runnable application checkout.

## Start an existing installation

Open PowerShell in the application folder:

```powershell
npm run dev
```

Open [localhost:3000](http://localhost:3000). Keep the terminal open. Stop with **Ctrl+C**.

## First-time application setup

Install Node.js 20.9+ and Python 3.10+. From the application directory:

```powershell
npm install
if (-not (Test-Path .env.local)) { Copy-Item .env.example .env.local }
```

A pnpm lockfile is also present in the application workspace. For a lockfile-based pnpm installation, use `pnpm install --frozen-lockfile` and the equivalent `pnpm` scripts consistently rather than mixing package managers.

Open `.env.local` in your editor. Keep this file local; do not upload it, paste its contents into chat, or rename secret variables to `NEXT_PUBLIC_`.

## Python flight worker

From the same application directory:

```powershell
py -3 -m venv work/flight-benchmark-venv
& '.\work\flight-benchmark-venv\Scripts\python.exe' -m pip install -r scripts/requirements-fast-flights.txt
```

No environment activation is needed. The Windows adapter defaults to that interpreter path. For another location or operating system, set `FAST_FLIGHTS_PYTHON` to the absolute path of the installed interpreter in `.env.local`.

```dotenv
FLIGHT_SEARCH_PROVIDER=fast_flights
DUFFEL_FLIGHT_SEARCH_ENABLED=false
DUFFEL_ALLOW_TEST_MODE=false
DUFFEL_LIVE_DATA_BLOCKED=true
```

The worker uses subprocesses, SQLite caching and outbound network access. Deployments need a Node.js server and Python support; an Edge-only runtime is insufficient. Requests do not create bookings, but returned search prices are not verified final totals.

## Gemini conversations

```dotenv
INTAKE_PROVIDER=gemini
GEMINI_API_KEY=replace_locally
GEMINI_MODEL=gemini-3.5-flash-lite
GEMINI_CHAT_MAX_PER_MINUTE=30
GEMINI_CHAT_MAX_PER_DAY=1000
GROQ_INTAKE_ENABLED=false
```

Use a Google AI Studio key with the intended project's permissions. Keep the project on its Free tier without attaching billing when free-only use is required. The application cannot verify billing status from the key. Local caps do not override Google's model/project quotas. See [Gemini details](GEMINI_CHAT.md).

For deterministic parsing without a hosted model, use `INTAKE_PROVIDER=local`. Groq is a separately configured alternative described in [Groq setup](GROQ_CHAT.md). Provider errors do not trigger an automatic switch.

## Hotels and hostels

The default is now key-free Xotelo (`LODGING_SEARCH_PROVIDER=xotelo`). No additional account is required for the supported city catalog. See [Free providers](FREE_PROVIDERS.md) for destination overrides, sampling, estimate limits, and current setup. The RapidAPI instructions below apply only if you deliberately select `LODGING_SEARCH_PROVIDER=rapidapi` again.

The implemented RapidAPI listing uses the following server configuration:

```dotenv
RAPIDAPI_BOOKING_HOST=booking-com15.p.rapidapi.com
RAPIDAPI_BOOKING_API_KEY=replace_locally
RAPIDAPI_BOOKING_ENABLED=true
```

Enable this only with the appropriate subscription and permitted quota. It is separate from Booking.com's official Demand API credentials and partner approval. Choose transport before searching the city stay dates, then select hotels, hostels or both. See [Accommodation planning](ACCOMMODATION_PLANNING.md).

## Trains and buses

The default is now `GROUND_SEARCH_PROVIDER=transitous`, with `TRANSITOUS_ENABLED=true`. No key or traveler age entry is needed for schedule-only searches. Select one city leg at a time. Shared-service usage requirements and lack of guaranteed ticket pricing are documented in [Free providers](FREE_PROVIDERS.md). The Tictactrip instructions below apply only when deliberately selecting that provider.

Live Tictactrip search requires approved partner access, `TICTACTRIP_API_TOKEN`, a verified `TICTACTRIP_PRICE_CURRENCY`, and `TICTACTRIP_GROUND_SEARCH_ENABLED=true`. Keep it disabled until authorized. Rail Europe ERA is a researched alternative, not a connected open fare feed. See [Ground transport](GROUND_TRANSPORT.md) and [Transit options](TRANSIT_SEARCH_OPTIONS.md).

## Database

The current trip UI stores ideas and selections in browser local storage. PostgreSQL is not required to try it. The Drizzle schema/migrations are available for database development, but database-backed trip persistence has not been demonstrated.

After deliberately configuring a local development `DATABASE_URL`:

```powershell
npm run db:generate
npm run db:migrate
```

## Troubleshooting

| Symptom | Action |
| --- | --- |
| `npm` is not recognized | Install Node.js and reopen PowerShell so its PATH is refreshed. |
| PowerShell blocks `npm.ps1` | Use `npm.cmd run dev` or `npm.cmd install`; no execution-policy change is necessary. |
| Port 3000 is already used | Check the terminal's reported URL, or run `npm run dev -- --port 3001` and open that port. |
| Missing Python / spawn error | Create the virtual environment and install its requirements; verify `FAST_FLIGHTS_PYTHON` if customized. |
| Chat quota/authentication error | Check the selected provider, key permissions and provider quota. Wait for the shown retry delay; the trip is preserved. |
| Airport lookup failed | Check the actual city/airport text and provider configuration. Google-mode airport resolution uses the local catalog; not every place has direct coverage. |
| A route has no priced options | Check the failed dated leg and exclusions. A complete itinerary needs every leg; try a different date, airport or city sequence. |
| Hotels missing | Confirm the RapidAPI host/subscription/quota and actual selected stay dates. Inspect per-city errors; a configuration badge does not prove live inventory. |
| Saved trips missing | Use the same browser/profile and site origin. Different ports/profiles have separate local storage. |
| Configuration changes do not apply | Stop the server with Ctrl+C, then run `npm run dev` again. |

Never clear browser storage as a routine troubleshooting step: it contains saved trip ideas and selections.
