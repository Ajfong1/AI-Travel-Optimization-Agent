# UI text style

- Use sentence case for headings, buttons, labels, and status badges: “Trip summary,” “Search again,” and “Live quotes.”
- Capitalize place names. Keep official airline/property branding and airport codes intact. Apply `formatPlaceName` at display boundaries, without changing search data or saved identifiers.
- Show dates as `Mar 3, 2027`, flexible months as `March 2027`, and times as `3:30 PM`. Preserve the airport’s local calendar date.
- Use `→` between route stops and date-range endpoints; use `·` between short metadata items.
- Use `countLabel` for quantities: “1 night,” “2 nights,” “1 flight option.”
- Short labels have no final period. Complete explanatory sentences and questions use normal punctuation.
- Keep essentials visible. Put longer explanations in expandable details rather than repeating them around the page.

Shared display helpers live in `src/lib/text-format.ts`. Flight dates delegate to the same formatter.
