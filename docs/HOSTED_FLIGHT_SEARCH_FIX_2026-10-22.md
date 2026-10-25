# Hosted multi-city flight search

Investigated the reported LA → Rome → Athens → Venice → LA search. Local airport-resolution regression checks select LAX → FCO → ATH → VCE → LAX, including all four journeys. These checks establish airport selection, not live fare availability.

The hosted configuration limits independent flight workers to one for the Free container's memory budget. The month-search scheduler still launched three date requests concurrently. While the first worker ran, other requests rejected immediately as busy and the scheduler advanced through unsearched dates, counting them as unavailable.

The date scheduler now receives the selected provider's concurrency limit. The independent provider advertises the same validated limit it enforces; Duffel retains its existing bound of three. Shared-worker contention pauses the month search instead of consuming all remaining dates. Exact-date requests return a specific busy response with a short retry hint. This does not increase the free server's worker or memory limits, enable paid providers, or add automatic retries to supplier requests.

Validation: 22 focused flight/provider tests passed, TypeScript passed, and targeted ESLint passed. The four-leg regression checks all 22 date combinations for a nine-night December trip with exactly one concurrent worker and no capacity failures. A second regression confirms contention stops before subsequent unsearched dates are marked failed. No booking or payment endpoints were used.

The capacity correction deployed successfully as `34806e1b769d917f0ae0f1ea2052f67bf62ec399` on Render Free. It addresses request scheduling after a route is accepted.

## City-combination rejection

The user supplied the actual error: **No city combinations fit the destination exclusions and required cities**, occurring across January–June. This error is thrown in the client before fare requests. Two reproducible structured-state bugs were found:

- Departure/return cities could also be stored in destination intents. The required-stop list then demanded LA as an additional visit while the candidate pool excluded endpoints, making every combination impossible.
- The chatbot could save confirmed named stops in `routeCities` without duplicating them in `destinations`. The candidate builder ignored that confirmed route and could produce an empty pool. Localized names such as Roma/Venezia also differed between candidate and requirement matching.

The candidate builder now recognizes confirmed route stops, normalizes the same city aliases used for airport matching, and removes endpoints from required sightseeing stops. Explicit city and country exclusions still apply. Regression tests reproduce the failures and verify that January–June dispatch the four legs LAX → FCO → ATH → VCE → LAX. They do not assume that changing dates resolves a city constraint error.

Validation after both corrections: **453 tests passed, one optional benchmark skipped**, TypeScript passed, and targeted ESLint passed. One bounded free Google search was also run locally for January 7/10/13/16, 2027: every leg returned five priced options, five complete independent-fare combinations were built, and elapsed provider time was 3.895 seconds. The lowest returned sum was USD 832, excluding unverified baggage. These dates were diagnostic examples, not an edit to the user's saved trip. No bookings, paid adapters or purchases were used.

The city-selection correction deployed successfully to the existing Render Free service as `9e30be63e410c8c9ca015c23a35ec8934e957787`; the Linux build and rollout completed in 1m24s. Post-deploy checks returned sign-in 200, the expected unauthenticated dashboard redirect and flight-lookup 401, and the expected same-origin validation responses. The user's exact saved dates and a live fare response from Render still need a signed-in hosted check. The local live diagnostic and regression tests do not establish datacenter fare availability or guarantee a particular price. Existing open pages need a full refresh to load the corrected client-side route selection.
