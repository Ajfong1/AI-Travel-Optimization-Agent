# Dashboard visual refresh

The interface now uses a teal brand with restrained coral, lavender and gold accents, a pale neutral background, and a custom SVG route/W mark. The same mark supplies the browser icon. Assets use local SVG/CSS and system fonts; no external font or image service is required.

The dashboard prioritizes transport and lodging, with budget and map cards alongside them on desktop. Trip settings, saved trips, selected items and detailed search conditions expand on demand. City stops form a compact horizontal route strip. The header provides six section links in a horizontally scrolling mobile row. Full city names remain in route cards and trip details; long itinerary headings are shortened.

Flight cards retain prominent total prices, traveler price basis, dates, departure time, flight duration and connection counts. Journey prices, airports, carriers and layovers remain expandable. Separate-ticket and unverified-baggage notices remain visible, while detailed provider conditions sit inside disclosures. Provider failures and stale-price warnings remain accessible.

The dashboard's **New trip** shortcut opens a fresh chat form without deleting previous trips. The **Edit trip** shortcut expands and scrolls to the existing edit form. Empty dashboards have a concise start action instead of multiple empty tools.

Validation: TypeScript and targeted ESLint checks; 20 existing pricing, journey, transport-plan, lodging-plan and flight-handoff tests passed. Browser review used an isolated local guest draft with model and paid-provider access disabled. Desktop was reviewed at 1280×900, mobile at 390×844; no page overflow was detected. Editing, transport tabs, section anchors and the new-trip shortcut were exercised. Flight-card expansion was visually checked using a temporary, clearly labeled local sample page, removed before publication. No sample trips were imported into a user's cloud account and no fare, hotel, booking or payment requests were needed for this visual review.

Screenshots remain under ignored `work/browser-check/`; they are not published as application data.
