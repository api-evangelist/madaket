---
name: madaket-nppes-lookup
description: Retrieve provider information from the federal NPPES registry through Madaket, and
  request a refresh of a provider's data from primary sources.
api: Madaket Provider API
version: v2.0
base_url: https://provider.madakethealth.com/provider-services
operations:
  - providerList1
  - providerAggregateSources
generated: '2026-08-25'
method: generated
source: openapi/madaket-provider-api.yml
---

# Query NPPES and refresh provider data through Madaket

## Look up NPPES

`POST /api/v2.0/provider/query` (`providerList1`) — *"Retrieve provider information from
NPPES"*. Send an `NppesPocProviderQuery` body; you get `NppesRecord` results back.

NPPES is the federal National Plan and Provider Enumeration System, the system of record for
the **NPI**. Use this when you want the enumerated federal record rather than Madaket's
aggregated view. Use `providerSearch` (see `skills/madaket-provider-lookup.md`) when you want
Madaket's own ranked database.

Madaket's companion open format, **ProviderJSON**
(`vocabulary/madaket-providerjson.md`), is explicitly modelled on the fields NPPES collects,
and switches its required-field sets on `enumeration_type`: `NPI-1` (individual), `NPI-2`
(organisation), `OEID` and `HPID`. Read that format if you need to know which fields are
mandatory for which enumeration type.

## Refresh a provider from primary sources

`POST /api/v2.0/aggregation-request` (`providerAggregateSources`) — *"request refresh of
Provider data from available primary sources"*. Body is an `AggregationRequestStub`; you get
back an `AggregationRequest`.

**This is the only operation in the entire 211-operation API that is not a read.** Treat it
accordingly:

- It **enqueues a job**. The response is a job record, not refreshed data. Re-read the
  provider afterwards to see the result.
- It is **not idempotent** — no `Idempotency-Key` or equivalent is defined. Calling it twice
  enqueues two jobs.
- **There is no cancel, abort or rollback**, and Madaket publishes no window in which one
  could be issued. Once submitted, it runs.
- Because of the above, an agent should **confirm with a human before calling it** rather than
  firing it speculatively or on retry.

## Everything else is read-only

The other 210 operations are GETs and query-by-POST reads. There is no create, update or
delete anywhere in this contract. Madaket's actual write surface — submitting and maintaining
payer enrollments — lives in the authenticated portal at
`https://enrollment.madakethealth.com/services/`, not in this API.

## Rules

- Authenticate with `api_key` + ten-minute `auth_token` in the query string; see
  `authentication/madaket-authentication.yml`.
- No pagination, no documented rate limits, no documented error envelope.
- The published base URL returned 503 on 2026-08-25. Verify liveness before depending on it.
