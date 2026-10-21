# Free lodging and transit backends

Implemented October 21, 2026. The default adapters are Xotelo lodging and Transitous public-transport schedules. Neither needs an API key. Existing RapidAPI/Booking Demand/Tictactrip integrations remain explicit alternatives; there is no paid fallback after a free-provider failure.

## Local configuration

```dotenv
LODGING_SEARCH_PROVIDER=xotelo
XOTELO_CITY_KEYS={}
GROUND_SEARCH_PROVIDER=transitous
TRANSITOUS_ENABLED=true
RAPIDAPI_BOOKING_ENABLED=false
TICTACTRIP_GROUND_SEARCH_ENABLED=false
```

These selections have been applied to the local development environment without removing old credentials. Restart `npm run dev` if environment changes are not picked up. No new account or key is needed for local prototype testing.

## Xotelo

The existing `/api/accommodation/search` endpoint uses the fixed public `https://data.xotelo.com/api/list` and `/rates` endpoints. It does not use the RapidAPI-only name-search endpoint. A small city-to-TripAdvisor-location-key catalog provides discovery. Localized city aliases are recognized. Unmapped cities return a specific setup message rather than searching another city.

For an additional city, set a lowercase city key in `XOTELO_CITY_KEYS`, for example `{"porto":"g189180"}`. Use the correct location's TripAdvisor `g` identifier, not an individual hotel's `d` identifier. The adapter validates key syntax, property key prefixes and property proximity to the selected arrival airport. This bounded catalog is not worldwide city coverage.

At most two list pages (200 properties) are examined per stay. Only explicitly classified hotels or hostels are retained; unknown categories are not guessed from property names. At most three properties per requested type receive dated rate queries. A hostel-only search never prices hotels. Results retain supplier attribution, retrieval timestamp and distance to the outgoing airport when coordinates exist.

Quotes are estimates using returned nightly room rates × requested rooms × nights. Occupancy is sent to Xotelo, but exact room/bed allocation, taxes and extras still need confirmation. Responses with wrong dates/currency or stale timestamps are rejected. Sampled search does not guarantee the cheapest property in a city; a missing hostel within the sample is not proof that no hostels exist. Chosen quoted stays still contribute to the estimated trip total, with charges flagged as unverified.

Reference: [Xotelo endpoint documentation](https://xotelo.com/).

## Transitous

`/api/ground/plan` resolves city/station names through `/api/v1/geocode`, then requests up to three itineraries through `/api/v6/plan`. Known European city-country matches prevent confusing Paris, France with US cities of the same name. Ambiguous matches ask for a more specific name.

One city leg is searched per request. Multi-city trips use the dashboard's leg buttons; automated region/month route permutations are not sent to the shared community service. Rail/bus exclusions apply to route queries; walking transfers, operator names, lines, duration and changes remain in the schedule. Cancelled/invalid itineraries are excluded.

Search starts at 08:00 UTC on the chosen date; displayed schedule times explicitly use UTC. Timetable coverage may not extend to far-future dates. No ticket prices are invented. Schedules are saved separately from priced transport offers and never enter cheapest-fare ranking or trip totals. They are visible in Trains & buses but are not sold or reserved in the app.

The shared service's [usage policy](https://transitous.org/api/) permits light prototype testing, requires noncommercial open-source use for ongoing operation, and asks users to contact the operators before potentially intensive use. This project's GitHub repository currently contains documentation only: before ongoing public deployment, publish the application under an appropriate open-source license and confirm suitability with the service operators. Commercial/heavier use needs a suitable independently operated backend; self-hosting [MOTIS](https://github.com/motis-project/motis) requires datasets, storage and runtime resources. This integration does not publish source or choose a project license for you.

Requests identify Wayfarer and its project URL. The dashboard links to [Transitous data sources](https://transitous.org/sources/) and [OpenStreetMap attribution](https://www.openstreetmap.org/copyright).

## Request controls

Both providers use fixed HTTPS endpoints, no redirects, no credentials and no automatic retries. A bounded process-local cache stores listing/geocode responses for one day, hotel rates for five minutes and schedules for fifteen minutes. Concurrent identical requests are deduplicated. Each provider host has a local cap of 30 uncached requests per minute; actual community/provider limits still apply. Cancellation/25-second request timeouts and the existing application API guard remain active. These caches/counters are per server process, not a distributed production quota system.

## Provider alternatives

Use `LODGING_SEARCH_PROVIDER=rapidapi` or `booking_demand` only deliberately with the separately authorized account settings. Use `GROUND_SEARCH_PROVIDER=tictactrip` only with approved partner credentials. Stored credentials do not select those providers automatically.
