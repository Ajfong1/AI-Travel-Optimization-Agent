# Accommodation planning

**Current default:** key-free Xotelo discovery and dated estimates now power the dashboard stay search. RapidAPI and Booking Demand remain explicitly selected alternatives, with no automatic paid fallback. See [Free providers](FREE_PROVIDERS.md) for scope, limits, and setup; earlier provider-specific sections below describe those optional adapters.

Accommodation search runs only after an `OptimizedRoute` is selected. `deriveAccommodationStays` converts each intermediate city visit into local check-in/check-out dates using an explicitly supplied IANA timezone. `buildAccommodationSearchRequests` builds per-city occupancy from trip travelers and explicit room assignments; if room assignments are omitted, it labels the one-room assumption in the plan.

Provider adapters implement `AccommodationProvider`. The planner checks the central provider registry before calling an adapter, checks date and room occupancy, validates the actual seller and referral URL using the shared trust policy, and preserves source and retrieval time. It displays only approved provider/seller booking links. Incomplete tax or mandatory fee data remains visible in the quote breakdown but produces no all-in total.

`priceAccommodation` takes per-room nightly rates, room count, tax inclusion/amount, and mandatory-fee inclusion/amount. It sums each room across the actual night count and adds only explicitly excluded charges, preventing double counting. Unknown charges, mismatched occupancy/dates, unavailable inventory, currency mismatch, or an incomplete night schedule prevent a complete total.

`compareFullTripEconomics` compares transport plus complete accommodation totals in one currency. It preserves the transport-first winner and can separately surface a materially cheaper complete-trip route, using a caller-supplied savings threshold.

## Dashboard search connections

Dashboard quotes can be added to the chosen trip, with one property selected per city visit; another property for the same dates replaces it. Selections survive reloads and can be removed. The trip budget sums selected transport and lodging quotes per currency as a **planning estimate**, explicitly identifying totals that include test flights and quotes needing refresh. It does not call these amounts verified final booking totals. Changing route dates, the selected flight route or incompatible accommodation preferences excludes outdated stays from the current total. Unselected stays, activities, transfers and undisclosed charges remain outside this subtotal.

Stay dates now also retain the actual outgoing journey's departure airport. New DataCrawler quotes use returned property coordinates and that airport's catalog coordinates to show straight-line miles/kilometers, not road distance or transfer time. Old quotes without coordinates show distance unavailable until refreshed. Arrival-airport distance is never silently substituted.

City matching accepts localized city names such as Milan/Milano, tolerates malformed irrelevant suggestions, filter groups and property entries, and accepts nullable optional labels/charges. It still validates the requested city, country, dates, currency and property filter. Airport resolution now matches both the city and expected European country rather than accepting the first fuzzy city suggestion (which previously selected Beaumont/Port Arthur, Texas, for Porto). New searches choose the matching Porto/OPO suggestion; old wrong-airport flight quotes need a fresh search. HTTP 429 is displayed as a quota/rate error, not as missing accommodation, and stops further supplier requests for the remaining cities in that search.

The dashboard supports **DataCrawler Booking COM on RapidAPI**, at the fixed host `booking-com15.p.rapidapi.com`. This subscription is separate from official Booking.com Demand API partner access. The RapidAPI adapter has been checked against an authorized account: destination lookup, the `ht_id` property-type filter catalog, and hostel price search returned live inventory for Paris in December 2026. Keys stay in server environment variables and never appear in client responses or URLs.

Configure `RAPIDAPI_BOOKING_API_KEY`, `RAPIDAPI_BOOKING_HOST=booking-com15.p.rapidapi.com`, and `RAPIDAPI_BOOKING_ENABLED=true` in `.env.local`. Restart if the development server does not reload the environment. Then choose a fresh flight itinerary, select hotels, hostels or both, enter residence/room criteria, and search in **Places to stay**. GET reports configuration readiness; an actual search verifies the current subscription, quota and inventory.

The adapter calls `searchDestination`, matches an exact city name and checks proximity to the arrival airport, then calls `getFilter` for the actual dates and occupancy. It obtains `property_type::…` tokens dynamically from the `ht_id` group. It never sends unfiltered shopping if a requested token is unavailable. `searchHotels` receives a single property-type token, `sort_by=price`, actual child ages, rooms, dates, residence location and display currency. Selecting both types runs two filtered searches; singular preferences run only the selected type. Results cover the first supplier page per type, with the five lowest returned disclosed totals saved per city. Property-type attribution follows the supplier filter, rather than guessing from property names. The provider's published `getSortBy` example identifies `price` as lowest-first.

Quotes preserve supplier source, room/dorm description and retrieval time. The comparison adds explicitly disclosed `excludedPrice` charges to `grossPrice` once, preserving both for display. It rejects mismatched currencies and stay dates; it does not infer missing taxes or claim an all-in final booking total. No property booking URLs, orders, automatic purchases or verified trip-budget totals are created. Failed/partial city searches remain visible. Authorization, quota, timeouts and malformed responses produce sanitized messages. Requests use a fixed HTTPS host, no redirects, cancellation and a 25-second timeout.

Provider documentation: [Search Hotels](https://rapidapi.com/DataCrawler/api/booking-com15/playground/Search%20Hotels), [Get Filter](https://rapidapi.com/DataCrawler/api/booking-com15/playground/Get%20Filter), [Get Sort By](https://rapidapi.com/DataCrawler/api/booking-com15/playground/Get%20Sort%20By).

## Official Demand API alternative

An alternative search adapter is implemented for Booking.com Demand API **3.2**, using its published OpenAPI description. The official provider research and registry still mark Managed Affiliate access as `access_pending`; that separate adapter stays unavailable until its own account access is configured and verified. This does not gate the user's separately configured RapidAPI subscription. Tests use isolated fixtures; fixtures are never displayed as live prices.

The dashboard's “Places to stay” panel follows a selected flight itinerary, preserving journey grouping so a connecting airport is not mistaken for an overnight city visit. It uses local arrival and departure dates, rather than assuming that an overnight flight arrives on its departure date. Hotels-only or hostels-only preferences are strict filters, not ranking hints. Changing lodging preferences preserves otherwise-current flight searches and selections; lodging quotes become stale.

`GET /api/accommodation/search` reports connection status. `POST` receives the selected itinerary, trip requirements, residence country/state, room count and display currency. It derives stays, requests the provider's property-type IDs once, resolves exact city IDs (airport-linked city lookup with an exact name match, or the configured ID mapping), and sends `filters.accommodation_types` plus `sort.by=price` / `direction=ascending` **before** shopping. It fetches details only for returned properties, verifies their property type, then keeps up to five lowest returned stay quotes per city. No unfiltered fallback is sent if a requested type or city cannot be resolved. The comparison covers the first 20 price-sorted properties, not all sites or guaranteed global minima.

These are supplier search quotes, not the older planner's fully validated room-by-room prices or approved booking offers. Search results preserve total/display-price semantics, flag potential conditional charges, and do not feed the verified trip budget. Property booking links/order creation are not exposed by this adapter. Account entitlements, room/dorm terms, fees, availability and seller/referral approval must be checked before a booking integration is enabled.

## Connect Booking.com

1. Obtain a [Managed Affiliate Partner agreement and Demand API access](https://developers.booking.com/demand/docs/getting-started/prerequisites).
2. Configure `BOOKING_COM_API_KEY`, `BOOKING_COM_AFFILIATE_ID`, and `BOOKING_COM_ACCOMMODATION_ENABLED=true` in the server environment. Never use `NEXT_PUBLIC_` or paste credentials into chat.
3. After account permissions are actually verified, update the `booking-com` provider's verification record in `src/server/providers/registry/config.ts` with that evidence. The existing `access_pending` state deliberately prevents calls even if credentials are present. Seller/referral approval is a separate step for booking links.
4. Resolve city IDs using `/common/locations/cities`. If the airport lookup does not unambiguously match the requested city, populate `BOOKING_COM_CITY_IDS` with a JSON mapping of lowercase city names to verified IDs. Do not silently search a nearby airport town instead.
5. Restart the app, choose a fresh flight itinerary, select hotels/hostels, supply residence/room criteria and run the stay search. Test authorization and market-specific price semantics before enabling production traffic.

The adapter uses the fixed production host `https://demandapi.booking.com/3.2`, server-only Bearer and `X-Affiliate-Id` headers, 25-second request timeouts, no redirects, cancellation, and the application's request guard. HTTP/provider errors are sanitized. It has not been validated against an authorized live lodging account in this workspace.

## Additional sources researched

### RapidAPI subscriptions

A Booking.com-branded API purchased through RapidAPI is a separate API contract from the official Demand API. Its exact listing/provider and `x-rapidapi-host` determine the endpoints, authentication and property-type filters. A RapidAPI authorization/application URL does not identify the subscribed API.

The DataCrawler listing is now implemented as described above. The status endpoint validates the configured host and key presence; it never sends a RapidAPI key to `demandapi.booking.com` or assumes official Booking.com partner access. Enter the key in local server configuration, not in chat or a `NEXT_PUBLIC_` variable.

- [Booking.com 3.2 filtering](https://developers.booking.com/demand/docs/accommodations/v3.2/filter-sorting) supports accommodation type and dormitory filters. The type filter lives inside `filters` in 3.2, unlike older 3.1 examples. [OpenAPI reference](https://developers.booking.com/demand/docs/open-api/3.2/demand-api).
- [Hostelworld Partner API](https://partner-api.hostelworld.com/) and [affiliate solutions](https://partners.hostelworld.com/solutions/) provide a potential second hostel source. Separate affiliate/API access and a provider adapter are required; it is not connected by this change.
- [Expedia Rapid](https://developers.expediagroup.com/rapid/lodging?locale=en_US) offers property content, geography and shopping APIs, with a later price-check step. It requires separate partner onboarding and integration. Hostel coverage and property-type catalogs need verification for the actual account before adding it to hostel-only searches.

Approved live offers are persisted in `accommodation_offers` with provider, source, retrieval time, exact stay dates, price completeness, seller approval evidence, normalized room quote, and validated booking URL. This table stores only offers that pass seller approval; rejected seller details remain in search diagnostics and should be handled under the application's logging/retention policy.
