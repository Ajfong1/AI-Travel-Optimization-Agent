# Larger routes, flexible city answers and baggage clarity

Updated October 21, 2026.

## City selection

The application no longer caps a trip at six destination cities or seven independently searched flight legs. Seven-city Gemini updates validate normally; explicitly listed cities are retained, including longer routes. Positive city counts still need enough trip nights and an adequate candidate pool. Unsupported regions or an insufficient curated city pool are explained rather than silently truncating a route or inventing cities.

“Any cities,” “any city,” “the biggest ones,” “major cities,” “I don’t know,” and similar city-choice replies delegate selection locally. They accept the draft route without a second city confirmation. South America now has an 18-city candidate pool, with country filters and Rio/Rio de Janeiro normalization. Its six-city example starts with Rio de Janeiro, Sao Paulo, Buenos Aires, Santiago, Lima and Bogota. These are curated major-city candidates, not a verified ranking of populations.

When all required details are ready, the existing loading indicator and one-time search handoff open the dashboard and launch flight comparisons. A delegated regional trip with a known starting city and no endpoint defaults to a disclosed return to the starting city. Existing endpoints and recorded one-way/open-jaw instructions are preserved. Unknown departure locations, dates, traveler ages and bag categories are still clarified.

The same local recovery supports saved drafts stuck behind the former city-choice loop. This does not reset model quotas or launch searches just by loading a saved trip.

## Larger-search safeguards

Removing the city cap does not mean launching every permutation at once. City orders are compared in batches of six or up to 24. BigInt permutation calculations prevent overflow; totals larger than JavaScript's exact numeric range are labelled as lower bounds. Required cities and exclusions remain part of candidate selection.

The flight worker searches all legs with two concurrent requests per worker. Its combination builder retains the five cheapest compatible paths per last-leg option, avoiding exponential enumeration. Its process timeout scales with leg count, bounded to 30 minutes; per-request timeouts, five-minute caching, cancellation and supplier availability still apply. Duffel's separate six-leg supplier limit remains explicit; the configured independent-leg provider supports larger routes. No paid fallback is enabled.

## Baggage

One personal item per traveler remains the default planning assumption. It is not represented as a verified airline allowance. Explicit personal-item changes, including zero, are preserved when checked-bag or carry-on answers are updated.

The chatbot asks one precise question at a time:

1. “How many checked bags (in the hold) per traveler? Enter 0 if none. We assume one personal item each.”
2. “How many carry-on bags (overhead bin) per traveler, in addition to one personal item each? Enter 0 if none.”

A bare count answers only the active named category. A bare count following an old vague “How many bags?” question is not assigned to a category; a precise clarification replaces that question. Explicitly named carry-ons are not rewritten as checked bags by frontend normalization. Generic model baggage prompts are replaced by the validated category-specific question.

## Verification

- 366 TypeScript regression tests across 54 files passed; 13 Python compatibility tests passed.
- Mocked Gemini accepts seven cities; a 12-city explicit route retains 13 flight slices. A 40-leg mocked worker test searches all 40 legs and builds five complete options. This is a correctness test, not a live performance benchmark.
- Large permutation tests unrank 25-city routes from 40 candidates without numeric overflow or lost mandatory cities.
- Local airport tests resolve all 18 South America candidates, including Montevideo/MVD and Sao Paulo/GRU. The live run exposed Montevideo's airport outside the city municipality; the adapter now uses explicit catalog-validated gateways.
- The user's saved six-city South America draft opened its dashboard and started April 2027 date comparisons from Denver. The first itinerary returned a lowest quoted USD 1,635 and a best-flight option at USD 1,701. After the airport fix, the resumed comparison finished with 10 checked city orders and 15 saved flight options. Prices can change or expire; availability and baggage costs remain unverified.
- All automated regression supplier/AI calls were mocked. Live verification used public Google flight searches through the configured worker; no paid Duffel requests, bookings or payments were made.

## Middle East city delegation (October 22, 2026)

Fixed a repeated city question after “whatever is the cheapest.” The local delegate matcher now accepts this wording, “whichever is the cheapest,” and “the cheapest cities,” records lowest-cost intent, and selects from a 16-city Middle East pool. Country/city exclusions and existing must-visit constraints still apply. Every candidate has a catalog-validated airport gateway, including airports outside the city municipality. The deterministic parser also recognizes the Middle East region and the new countries.

An unsupported area now produces a specific scope clarification with an honest catalog limitation instead of a model promise followed by the same city-list question. Missing required details such as carry-on counts are still asked one at a time. Once complete, the existing automatic handoff opens the dashboard and starts the bounded route comparison. Cheapest means cheapest returned eligible results in the searched batch, not a guarantee over every regional permutation.
