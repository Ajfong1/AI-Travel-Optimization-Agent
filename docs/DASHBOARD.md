# Trip dashboard

The dashboard opens at `/dashboard`. A completed planner conversation automatically opens `/dashboard#transport` with a one-time search handoff. Manual Search flights, paused/resumed comparisons and saved results remain available. Trip state and selections are held in this browser's local storage.

| Section | Purpose |
| --- | --- |
| Overview | Dates, route summary, search status and editable requirements |
| Chosen trip | Selected transport and the plan being assembled |
| Transport | Settings, progress, expandable flight combinations and ground options |
| Hotels & hostels | City stay searches, type filters and chosen lodging |
| Trip total | Compatible selected transport/lodging subtotal by currency |
| Map | Ordered route preview or selected journeys using available coordinates |
| Schedule | Selected transport in trip order |
| Compare routes | Compact city-itinerary summaries and flight alternatives |

Expand flight cards for returned airline names, airports, dates, times and layovers. Direct flights only applies to every leg. City comparisons show compact lowest-fare and shortest-flight summaries; selecting an itinerary exposes fuller flight combinations.

The map follows each journey's endpoints rather than drawing every flight from the origin. Missing coordinates remain a limitation. Property-to-airport distance is straight-line distance, not transfer time.

Choose transport before searching its city stay dates. Add a hotel/hostel to include that quote in the subtotal; another property for the same visit replaces it. Route/date changes can invalidate selections. Currency totals remain separate; missing charges are not silently included.

Saved trips are available in the main navigation and dashboard manager, with filtering, deletion, Undo and recovery. They are not synced to an account or database. No booking/payment is exposed. See [User guide](USER_GUIDE.md) and [Current status](CURRENT_STATUS.md).
