# Provider registry and seller policy

The provider registry is a server-side policy boundary, independent of any one transport search API. Provider facts are curated in `src/server/providers/registry/config.ts`; types, runtime validation, and seller/link checks live beside it.

## Fail-closed defaults

The initial registry contains researched provider definitions but an empty seller-approval list. Documentation review, a consumer website, or a supplier returned by a search API does not grant recommendation or booking-link approval. Until the organization has reviewed evidence and granted access, these providers cannot pass seller policy.

Each provider record stores its canonical ID, legal/company name, aliases, product and seller roles, official domains, capabilities, geography/inventory claims, verification evidence/date, handoff methods, and restrictions. The approval policy is independently configured for airlines, flight agencies, hotels, hostels, rail operators, bus operators, and activity ticketing providers.

## Granting or revoking approval

Add an approval record only after documenting the applicable agreement, API/referral access, seller identity, permitted handoff, official domains, and review date. Use the provider's canonical ID and a category-specific record. Set a finite `expiresAt` where the agreement or verification requires it. To revoke an approval, append a newer record with `status: "revoked"`; latest review wins, so an older approved entry cannot override revocation. Expired, pending, missing, stale, or revoked records fail closed.

No credentials or API secrets belong in this registry. Keep those in server-only environment configuration and provider adapters. Commercial terms and access are not inferred from this file; current status and evidence must be reviewed by an operator before configuring an approval.

## Applying the policy

Before an option can be recommended for booking, call `validateSellerEvidence` with the search source, product, actual seller ID and name, seller role, operating carrier where applicable, handoff method, and booking URL. Use `filterRecommendedOffers` to retain rejected offers with a reason for audit/debugging. Apply `prioritizeDirectAirline` only to suitable, validated options and only when the equivalence key represents the same itinerary, fare conditions, baggage, and comparable total.

URL validation requires HTTPS, rejects credentials, local/IP/IDN hosts, look-alike or unapproved domains, and detects explicit redirect destinations in URL parameters. The validator does not fetch links: following arbitrary URLs server-side could create an SSRF risk. Provider-specific redirect links without a verifiable final destination must be resolved through documented provider integration or excluded; a domain allowlist by itself is not proof of supplier identity or booking readiness.

The registry does not mark offers as repriced or booking-ready. Supplier offer status, baggage-inclusive total, inventory, and revalidation before handoff remain responsibilities of the offer validation and booking-handoff layers. Never describe a policy-approved link as guaranteed bookable.
