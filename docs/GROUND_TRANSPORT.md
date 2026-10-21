# European rail and bus transport

The shared transport contract now supports flight, train, bus, and mixed-mode legs. The first ground adapter is Tictactrip, the partner API identified in [provider research](./PROVIDER_RESEARCH.md). The documented API provides `GET /v2/autocomplete` and `POST /v2/results`; the adapter resolves unambiguous stop IDs and maps only the documented result schema. Search is disabled by default and requires an authorized partner token.

See the [official API reference](https://developers.tictactrip.eu/api), especially [`GetAutocomplete` and `GetResults`](https://developers.tictactrip.eu/api), and the [official search guide](https://developers.tictactrip.eu/docs/search). The search response documents itinerary and segment prices in cents; the official [glossary example](https://developers.tictactrip.eu/docs/glossary) presents one fare in euros. However, the documented result schema does not return a currency field or state a universal currency for every partner account. Therefore `TICTACTRIP_PRICE_CURRENCY` must remain unset until the provider confirms the account currency; without it the adapter retains the numeric `priceCents` but marks the offer incomplete and cannot rank it. The response also exposes UTC epoch timestamps, local ISO timestamps and UTC offsets, stop IDs/names, operator, transport type, duration, availability flags, coarse baggage inclusion, fare information, and redirect references.

## Configuration and API

Set server-side `TICTACTRIP_API_TOKEN` and explicitly set `TICTACTRIP_GROUND_SEARCH_ENABLED=true` only after the provider has approved access and commercial terms. Set `TICTACTRIP_PRICE_CURRENCY` only after the price currency is confirmed in the partner contract or technical documentation. The server endpoint is `POST /api/ground/search` with:

```json
{
  "origin": "Paris",
  "destination": "Brussels",
  "departureDate": "2026-12-11",
  "passengerAges": [30],
  "modes": ["train", "bus"]
}
```

`originId` and `destinationId` may contain a previously selected Tictactrip `stopCluster` or `stopGroup` ID. Free-text place lookup is accepted only when autocomplete yields one exact match or a single result; ambiguous names return `unresolved_location`. A provider no-result response is reported as `no_inventory` for that provider query, not as proof that no transport service exists.

The adapter has a 12-second per-request timeout, at most three attempts for transient network/429/5xx failures, no response logging, fixed provider origin, redirect rejection, and process-level API rate limiting. It does not implement purchasing or ticket issuance. Search results carry `search_result_retrieved`; search-time availability is not treated as reprice or booking readiness.

## Safety and coverage limits

- Only the provider-returned European inventory is searched. The API docs state a request may be ignored beyond 1,800 km; route availability and operator coverage are account/inventory dependent.
- Tictactrip documents `includedBaggage`, but not enough bag-type/count/weight/dimension/fee detail for the baggage-pricing engine. An offer cannot enter fully verified-cheapest ranking until these details are separately verified and passed through `priceTransport`.
- The public result has no currency property. A glossary example shows euros, but this is not enough evidence to assume currency for every account or market. Missing configured/confirmed currency leaves the fare incomplete and out of cheapest ranking.
- `redirectionLink` is not exposed as `bookingUrl`: the project has no active Tictactrip seller approval, and the result may redirect to a third-party carrier. The offer/segment IDs are preserved as references. Do not use search data to bypass the seller registry.
- Tictactrip documents multimodal itinerary assembly, but public docs do not confirm protected connections or segment minimum connection times. The adapter marks connection protection unknown. `validateTransportRoute` can reject impossible/short connections, detect overnight legs, flag independently ticketed connections, and require transfer-time evidence for station/airport changes. Its 30-minute default is a planning screen only, not a supplier guarantee.
- The API exposes local offset information but not a named IANA timezone in the result example. The normalized leg retains offsets; do not imply an IANA timezone was resolved.
- Provider date inventory gaps, presale periods, unavailable segments, or malformed schedules are represented as incomplete/unavailable/no-inventory states and never filled with synthetic live results.

The adapter tests use synthetic responses modeled on official examples and mocked fetch. They do not prove current production credentials or inventory access.

## Planner and dashboard integration

The planner now calls `POST /api/ground/plan` after its flight search, including when a flight search fails. If flights are excluded, it skips them and still attempts ground search. Completed results and availability explanations are saved with the originating trip. A month flight comparison supplies the first shortlisted departure date for the ground query; this does not claim a whole-month ground fare comparison.

The dashboard's **Trains & buses** tab accepts city/station names, an exact date, and actual traveler ages for any part of the trip. This allows separate ground legs to be searched after a flight into Europe. It respects excluded train/bus modes. A configured Tictactrip account searches returned train, bus and mixed combinations through the existing adapter. Without partner credentials, the application records an explicit unavailable state with no invented quotes. Ground fares remain separate from baggage-inclusive flight rankings because baggage fees are not fully verified, and one-way ground fares are not presented as equivalent to complete round-trip flights.

Rail Europe also publishes an [ERA API developer entry point](https://raileurope.github.io/era-api/) and offers [partner distribution](https://agent.raileurope.com/). It is an additional commercial integration option; the application does not currently implement ERA or claim access to it. Tictactrip's [official developer platform](https://developers.tictactrip.eu/) explicitly requires API access on request. No new credentials, commercial accounts, or carrier permissions were created during this implementation.

## Chosen transport and route map

Flight, train and bus options can be added to **Chosen trip**. Choices are stored per trip, survive reload/deletion recovery, and retain test/live mode, quote currency and requirement provenance. A new option with the same endpoints replaces its previous alternative. Separate selected legs form a departure-ordered transport schedule. Changed requirements or expired offers require refresh; overlapping selected journeys and unverified station/airport transfers are flagged. Selecting transport never creates a reservation or payment.

The map resolves exact IATA coordinates from the supplied airport catalog and can consume provider-supplied stop coordinates retained in the display schema. It does not substitute nearby airports for rail/bus stations. Unresolved stations are listed rather than plotted at invented coordinates. With no selection, it previews the first returned option; with selections, it shows the chosen route. Blue/purple/orange lines indicate flight/train/bus and are schematic connections, not actual airways, rail tracks or roads. Test inventory stays labeled.

The basemap uses ordinary viewport-only OpenStreetMap raster requests with visible attribution and browser caching. Review the [OSM tile policy](https://operations.osmfoundation.org/policies/tiles/) before production traffic. No offline download or tile prefetch feature is provided. The map remains a best-effort service; failures are shown while known coordinates and connections remain available. The airport catalog's source/license caveats in `AIRPORT_CATALOG.md` still apply.

## Saved trip management

Saved trip ideas are collapsed by default and support name filtering, single deletion, bulk selection/deletion, Undo, and restoration from **Recently deleted**. Recovery data stays in this browser until browser storage is cleared. Deleting the active trip selects another remaining trip or clears the current draft. Earlier session-only drafts cannot reappear after the last saved trip is deleted.
