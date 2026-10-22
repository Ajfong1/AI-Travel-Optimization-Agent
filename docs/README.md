# Documentation index

**Current snapshot: October 22, 2026.** This publication contains documentation; application source and ignored local evidence files are not included. Source references describe the complete development workspace.

## Start here

- [Agent efficiency review and four-trip benchmarks](EFFICIENCY_BENCHMARK_2026-10-22.md)

- [Larger routes, flexible city selection and clear baggage questions](ROUTE_AND_BAGGAGE_FIXES.md)

- [Chat, Boston stations and flight-search fixes](CHAT_AND_BOSTON_CHECK_2026-10-21.md)

- [Edit/transit bug fixes, Airbnb research and benchmark](EDIT_AND_TRANSIT_CHECK_2026-10-21.md)

- [Project README](../README.md)
- [Local setup and PowerShell commands](LOCAL_SETUP.md)
- [User guide](USER_GUIDE.md)
- [Current implementation and verification](CURRENT_STATUS.md)
- [Dashboard](DASHBOARD.md)

## Integrations and behavior

- [Mixed flights, trains and buses](MIXED_TRANSPORT.md)

- [Free Xotelo and Transitous backends](FREE_PROVIDERS.md)

- [Gemini conversations](GEMINI_CHAT.md)
- [Groq conversations](GROQ_CHAT.md)
- [Independent flight-leg search and benchmarks](INDEPENDENT_LEG_SEARCH.md)
- [Airport catalog](AIRPORT_CATALOG.md)
- [Accommodation and RapidAPI](ACCOMMODATION_PLANNING.md)
- [Ground transport setup](GROUND_TRANSPORT.md)
- [Transit research, including Rail Europe ERA](TRANSIT_SEARCH_OPTIONS.md)
- [UI text conventions](UI_TEXT_STYLE.md)

## Engineering designs and domain rules

These describe domain modules, proposed architecture, or development policies; they do not prove that every workflow is connected to live suppliers.

- [Technical architecture](TECHNICAL_ARCHITECTURE.md)
- [Data model](DATA_MODEL.md)
- [Route optimization](ROUTE_OPTIMIZATION.md)
- [Alternative ranking](ALTERNATIVE_RANKING.md)
- [Baggage pricing](BAGGAGE_PRICING.md)
- [Trip editing](TRIP_EDITING.md)
- [Activity itineraries](ACTIVITY_ITINERARIES.md)
- [Provider registry](PROVIDER_REGISTRY.md)
- [Provider research](PROVIDER_RESEARCH.md)
- [Booking handoff design](BOOKING_HANDOFF.md)
- [Duffel integration and cost block](DUFFEL_FLIGHTS.md)

## Historical checks and operational notes

- [October 20 four-trip browser check](TRIP_FLOW_CHECK_2026-10-20.md): dated observations, not current fares.
- [Native multi-city investigation](FAST_FLIGHTS_MULTICITY.md): superseded by independent-leg search.
- [Testing audit](TESTING_AND_VALIDATION.md): original October 18 audit; see Current status for later checks.
- [Security review](SECURITY_REVIEW.md): historical review and remaining production work.
- [Deployment](DEPLOYMENT.md): original operational notes; use Local setup for current development instructions.

Raw benchmark JSON, screenshots, quota ledgers, Python environments and API keys are deliberately not published. Historical `work/` links refer to local evidence, not files included in GitHub.
