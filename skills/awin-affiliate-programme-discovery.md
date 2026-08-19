---
name: awin-programme-discovery
description: Find advertiser programmes a publisher can join or is already in, read their detail records, and check the commission rates before committing promotional effort.
api: openapi/awin-affiliate-programmes-api-openapi.yml
operations:
  - getAccounts
  - getProgrammes
  - getProgrammeDetails
  - getCommissionGroups
generated: '2026-08-13'
method: generated
source: openapi/_original/awin-affiliate-openapi.yml + https://help.awin.com/apidocs/promotions
---

# Discover and qualify Awin advertiser programmes

Use this when a publisher asks "what should I promote?" or "what does this brand actually pay?"

## Before you call

Bearer token from <https://ui.awin.com/awin-api>, `Authorization: Bearer <token>`.
These are all reads and safe to retry.

## Steps

1. `getAccounts` (`GET /accounts`, `type=publisher`) → resolve `publisherId`.

2. **List programmes** with `getProgrammes`
   (`GET /publishers/{publisherId}/programmes`).
   - `relationship` is the important filter — it separates programmes the publisher has
     **joined** from ones they have **not joined**, plus pending and suspended states. Asking
     "what could I join?" and "what am I in?" are two different calls, not a client-side filter
     on one call.
   - Add the country filter only when the publisher's audience is country-specific.

3. **Read the detail** for a shortlisted programme with `getProgrammeDetails`
   (`GET /publishers/{publisherId}/programmedetails`).

4. **Check the money** with `getCommissionGroups`
   (`GET /publishers/{publisherId}/commissiongroups`) before recommending anything. Commission
   groups are the actual rate structure — a programme with one DEFAULT group behaves very
   differently from one that splits rates by product category.

## Rules you must not break

- **20 calls per minute per user**, no rate-limit headers. If you are qualifying a long
  shortlist, serialize and pace; do not fan out one detail call per programme in parallel.
- **Do not present a commission rate as a promise.** Rates change in the platform and this API
  is a read of current state, not a contract. Say when the data was read.
- **Currency belongs to the programme**, not to the publisher. Show it.
- Programmes the publisher has not joined may return thinner detail records — report the gap
  rather than filling it in.

## Errors

See `errors/awin-affiliate-problem-types.yml`. `404` on programme details usually means the
programme id is not visible to this publisher relationship, not that it does not exist.
