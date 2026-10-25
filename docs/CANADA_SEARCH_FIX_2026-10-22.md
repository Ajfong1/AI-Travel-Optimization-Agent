# Banff and Vancouver search corrections

The user reported Vancouver lodging failing on LA → Banff → Vancouver → LA. Flight lookup and lodging are separate flows; the hotel result is not a prerequisite for airfare searches. The lodging endpoint already loops over each stay independently.

Three reproducible issues were corrected:

- Vancouver was absent from the free Xotelo destination map. Added `g154943`, verified against [TripAdvisor's Vancouver destination](https://www.tripadvisor.com/Hotels-g154943-Vancouver_British_Columbia-Hotels.html) and a live Xotelo response. No API key or paid fallback is needed.
- Names such as `Vancouver, BC` and `Vancouver, British Columbia, Canada` did not resolve to the Canadian city's airport or lodging key. They now normalize to Vancouver; Vancouver, Washington is kept distinct.
- Flexible city comparisons could drop explicitly named towns outside the automatic major-city dictionary. Confirmed named towns now remain in the candidate universe, including Banff. Explicit city/country exclusions and departure/return exclusions still apply. Banff retains Calgary (YYC) as its flight gateway and a separate ground transfer.

Regressions cover airport resolution of the three flight legs LAX → YYC → YVR → LAX without hotel requests, Vancouver's lodging mapping, both Canada route orders, country exclusions, and the full lodging API with Vancouver as stay two. Vancouver still returns its own lodging results when Banff's provider request fails.

A bounded live free-provider probe sampled 200 Vancouver listings: 71 explicitly classified hotels and 14 hostels. Dated rate responses were returned for English Bay Hotel and Samesun Vancouver, January 10–13, 2027, one room/two adults/USD. These were diagnostic dates, not changes to the user's trip. Sampling is limited to 200 listings and three priced properties per requested type in the application; it is not a city-wide cheapest guarantee. Taxes, room allocation and availability remain subject to the provider's limitations. No property was reserved or purchased.

Validation: 458 regression tests passed, one optional benchmark skipped, TypeScript passed and targeted ESLint passed. The hosted saved-trip UI could not be inspected because the local browser was signed out. The provider diagnostic and lodging API regressions verify distinct parts of the flow; they do not establish the user's exact-date availability from the Render server.
