---
name: madaket-provider-lookup
description: Find a US healthcare provider in Madaket's database and load their full record,
  starting from a name, NPI or license and ending with a Madaket GUID and full provider context.
api: Madaket Provider API
version: v2.0
base_url: https://provider.madakethealth.com/provider-services
operations:
  - providerSearch
  - providerList
  - providerLoad
  - providerLoadFullContext
generated: '2026-08-25'
method: generated
source: openapi/madaket-provider-api.yml
---

# Look up a provider in Madaket

**Before you start — read this.** Madaket's published base URL returned HTTP 503 on every
path probed on 2026-08-25 (see `lifecycle/madaket-lifecycle.yml`). This skill describes the
contract as Madaket published it, reconstructed from Madaket's own npm SDK. Confirm the host
is answering before relying on it.

## Authenticate

Both credentials go in the **query string**, not in headers.

- `api_key` — issued by Madaket on request. There is no self-service signup.
- `auth_token` — base64url( SHA-256( `api_key` + `api_secret` + `timestamp` ) ), where
  `timestamp` is the current GMT time formatted `yyyy-mm-dd-HH-MM` **with the final character
  removed**, producing a rolling ten-minute window.

Recompute `auth_token` at least every ten minutes; do not cache it. Because the credentials
are in the URL, treat the whole request URL as a secret and never log it.

## Steps

1. **Search by what you know.** `POST /api/v2.0/provider/search` (`providerSearch`) with a
   `ProviderSearchQuery` body. It returns **ranked** results. Useful fields: `npi`,
   `firstName`, `middleName`, `lastName`, `phone`, `dob`, `ssn`, `providerTypes`,
   `taxonomies`, `licenseStates`, `licenseNumber`, `maxResults`.
   - If you have an exact NPI, search on `npi` alone — it is the strongest identifier.
   - If you only have a name, always add a state or taxonomy. Names are not unique.
2. **Or filter for an exact set.** `POST /api/v2.0/provider/filter` (`providerList`) with a
   `ProviderFilterQuery` body when you want matching rows rather than a ranked list.
3. **Take the `id` from the result.** That is the Madaket GUID, and it is the key every other
   resource in this API is addressed by (as `providerGuid`).
4. **Load the record.** `GET /api/v2.0/provider/{id}` (`providerLoad`) for basic identity, or
   `GET /api/v2.0/provider/{id}/full` (`providerLoadFullContext`) for the full
   `ProviderContext` — use the full form when you are going to need more than name and NPI.

## Rules that will bite you

- **There is no pagination.** No cursor, offset, page or limit parameter exists anywhere in
  this API. `maxResults` caps one response and cannot walk a result set. **Never assume a
  response is complete** — if you suspect truncation, narrow the query instead.
- **Handle 401 as a clock problem first.** The most common failure with this scheme is an
  `auth_token` computed from a mis-formatted or un-truncated GMT timestamp, not a bad key.
- **A 503 is the host, not you.** See `errors/madaket-problem-types.yml`.
- Madaket publishes no rate limits and no 429 semantics. Be conservative and serialise.

## Related

- `skills/madaket-credentialing-verification.md` — pull licences, education and sanctions once
  you have the GUID.
- `skills/madaket-nppes-lookup.md` — go to the federal NPPES registry through Madaket instead.
