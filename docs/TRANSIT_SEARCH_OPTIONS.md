# Transit search options

**Implemented update, October 21, 2026:** Transitous now supplies free city-to-city schedule options through the dashboard's Trains & buses search. It searches one explicit dated leg at a time and preserves walking transfers/operators. Ticket prices remain unknown and excluded from totals. See [Free providers](FREE_PROVIDERS.md) for configuration, live verification and shared-service usage requirements. Tictactrip/ERA remain optional partner alternatives.

Reviewed October 20, 2026. Train and rail are the same transport mode in this app; bus is a separate mode.

## Available now without a paid API

Every between-city leg has a Google Maps transit-view link in the planner and the dashboard's Trains & buses tab. Links carry the city endpoints and `travelmode=transit`. The user sets the departure date/time in Maps; the supported Maps URL API has no departure-date parameter. These links retrieve no fares in the app and do not contribute invented prices to the trip total.

Google's [Maps URL documentation](https://developers.google.com/maps/documentation/urls/get-started) states that an API key is not required. Opening the link is separate from using the paid Google Routes API.

## Searchable fares

- [Tictactrip](https://developers.tictactrip.eu/): European train and bus inventory, including combinations. The app already has an adapter and actual-age/date search form, and can add returned offers to a chosen trip. Partner authorization is required. No credentials are configured or supplier calls made as part of this change. Confirm pricing and search rights before enabling it.
- [Rail Europe ERA](https://docs.era.raileurope.com/): European rail search; not a bus source. [Rail Europe's setup guide](https://raileurope.github.io/era-api/postman/) requires credentials provided by Rail Europe. A separate adapter and commercial access would be needed.
- [Transitland routing](https://www.transit.land/documentation/routing-platform/): GTFS-based transit schedules and routing can help compare journey times and transfers. Schedule data is not evidence of a currently purchasable intercity ticket or its cheapest fare. Coverage, access requirements and costs must be checked for the intended regions.

## Next integration boundary

Keep searches per dated city leg. Compare the same endpoints, dates, passengers and currency. Keep fare sums, baggage/transfer charges and door-to-door travel times explicit. Schedule-only routes must retain an unknown price and cannot win a cheapest-price comparison. An unavailable ground provider must not cause an otherwise valid flight search to fail. No automatic reservations or payment calls belong in this planning flow.

The four flight benchmarks in `work/flight-benchmark/independent-legs/report.md` do not include a live ground-fare benchmark, because no authorized, confirmed-no-charge rail/bus fare account is configured.
