# AI Travel Optimization Agent

**Wayfarer** is a conversational travel planner for comparing flights, building multi-city routes, and organizing selected transport and accommodation in one dashboard.

> This GitHub repository currently publishes the project's documentation. The application source, Python scripts and dependency lockfile remain in the local development workspace; this documentation upload does not publish them. Run commands below from a complete application checkout, not a docs-only clone.

## What it does

- Understands conversational trip ideas, including vague regions, flexible months, trip length and multi-city routes without the former six-city cap.
- Asks one question at a time. “I don't know” about cities or moving pace delegates flexible route comparison without repeating the same question.
- Starts flight search automatically when required trip details are complete, shows a loading indicator, and opens the dashboard's Transport section.
- Searches every flight leg independently, including the middle legs of multi-city trips. Shows actual returned airline names, dates, times, airports, layovers and direct-flight filtering.
- Compares city itineraries with compact cheapest/shortest-flight summaries, then exposes the fuller flight combinations for a selected itinerary.
- Saves trip ideas in this browser, with a dedicated page for accessing, deleting and recovering them.
- Builds a chosen trip from transport and lodging selections, with a route map, schedule and estimated total by currency.
- Searches hotels and hostels through the configured RapidAPI adapter; a single-type preference restricts the search. Airport distance uses available coordinates and is straight-line distance.
- Supports partner-gated train and bus adapters and external transit search links. Live ground fares require approved provider access.

## Run locally in PowerShell

Requirements: Node.js 20.9 or later; Python 3.10 or later for the Google Flights worker. PostgreSQL is optional for database development; the browser planner does not require it.

From the complete application folder:

```powershell
npm install
if (-not (Test-Path .env.local)) { Copy-Item .env.example .env.local }
npm run dev
```

Open [localhost:3000](http://localhost:3000). Leave PowerShell running; press **Ctrl+C** to stop. If the application is already installed, only `npm run dev` is needed. The existing project also includes a pnpm lockfile and supports `pnpm install` / `pnpm dev`; use one package manager consistently.

Flight search additionally needs the isolated Python environment. See [Local setup](docs/LOCAL_SETUP.md) for the complete commands and environment settings. Never overwrite an existing `.env.local` when following setup instructions.

## Configure the chatbot

Keep credentials in `.env.local` on the server:

```dotenv
INTAKE_PROVIDER=gemini
GEMINI_API_KEY=your_google_ai_studio_key
GEMINI_MODEL=gemini-3.5-flash-lite
GROQ_INTAKE_ENABLED=false
```

Restart the development server after configuration changes. Gemini is the current configured chatbot provider; Groq and a deterministic local parser are alternatives. Model updates are validated before changing a trip. Errors preserve the draft, and there are no automatic model/provider retries or fallbacks.

Google's project quotas still apply. The application's Gemini safeguards default to 30 requests per rolling minute and 1,000 per Pacific calendar day; these are not promised Google allowances. A free-tier model does not guarantee free usage on a project with billing enabled. See [Gemini configuration](docs/GEMINI_CHAT.md).

## Flight search and cost controls

The local backend now defaults to key-free Xotelo lodging and Transitous train/bus schedules. The previous subscription adapters are opt-in. See [free-provider setup and limitations](docs/FREE_PROVIDERS.md), including Transitous's noncommercial/open-source usage requirements. Schedule-only results never count as free tickets or enter trip totals.

The local flight integration uses [AWeirdDev/flights](https://github.com/AWeirdDev/flights) through a Python compatibility layer. Each dated leg is searched as a one-way journey; complete options require results for every leg. Native Google multi-city search is retained for diagnostics rather than used by default.

Keep these settings for the current local search path and blocked Duffel live access:

```dotenv
FLIGHT_SEARCH_PROVIDER=fast_flights
DUFFEL_FLIGHT_SEARCH_ENABLED=false
DUFFEL_ALLOW_TEST_MODE=false
DUFFEL_LIVE_DATA_BLOCKED=true
```

Separate-ticket fares are planning quotes. Baggage, transfers, connection protection, taxes and availability may need further verification. The app does not guarantee the globally cheapest trip. Search coverage, failures and stale results matter; exhaustive city/month comparisons can be very large.

**No flight booking, reservation or payment flow is implemented.** Provider subscriptions may have their own terms and charges; enabling an adapter is not proof of free access. Never publish real keys or use `NEXT_PUBLIC_` for provider secrets.

## Documentation

Start with the [documentation index](docs/README.md):

| Guide | Contents |
| --- | --- |
| [Local setup](docs/LOCAL_SETUP.md) | PowerShell commands, Python worker, environment variables and common errors |
| [Using the planner](docs/USER_GUIDE.md) | Chat, automatic search, comparisons, saved trips and chosen plans |
| [Current implementation](docs/CURRENT_STATUS.md) | Runtime architecture, supported features, limitations and recent verification |
| [Dashboard](docs/DASHBOARD.md) | Navigation, transport, accommodation, map and totals |
| [Independent flight legs](docs/INDEPENDENT_LEG_SEARCH.md) | Worker design, airport lookup, multi-city behavior and benchmark observations |
| [Accommodation](docs/ACCOMMODATION_PLANNING.md) | RapidAPI Booking.com, hotel/hostel filtering and official Demand API distinction |
| [Ground transport](docs/GROUND_TRANSPORT.md) | Tictactrip setup and authorization requirements |
| [Transit research](docs/TRANSIT_SEARCH_OPTIONS.md) | Rail Europe ERA and other possible integrations |

Older engineering audits and design proposals are retained for context. The index identifies historical material; [Current implementation](docs/CURRENT_STATUS.md) takes precedence for today's behavior. Some technical docs reference local source paths or ignored benchmark artifacts that are not part of this documentation publication.

## Development checks

From the application checkout:

```powershell
npm run typecheck
npm run lint
npm test
npm run build
```

The latest recorded October 21, 2026 check passed 337 Vitest tests, TypeScript and ESLint for the changed automatic-search files. A browser check submitted a complete LAX–SFO round trip, opened the dashboard automatically and returned five flight options. This is development verification, not a production certification or booking guarantee.
