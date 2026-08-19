---
name: awin-advertiser-performance-reporting
description: Read an advertiser's Awin programme performance by publisher, campaign and creative, including the plan entitlement that gates advertiser API access at all.
api: openapi/awin-affiliate-reports-api-openapi.yml
operations:
  - getAccounts
  - getAdvertiserPublisherReport
  - getAdvertiserCampaignReport
  - getAdvertiserCreativeReport
generated: '2026-08-13'
method: generated
source: openapi/_original/awin-affiliate-openapi.yml + https://help.awin.com/apidocs/reports
---

# Report Awin advertiser programme performance

Use this to answer "which partners are actually driving my programme, and on what creative?"

## Before you call

- Bearer token from <https://ui.awin.com/awin-api>, `Authorization: Bearer <token>`.
- **Entitlement gate.** Awin states plainly that "API access for Advertisers is limited to
  Accelerate and Advanced plans only." If the advertiser is on a lower tier, a `403` is the
  commercial answer, not a technical failure — say so rather than retrying.

## Steps

1. `getAccounts` (`GET /accounts`, `type=advertiser`) → resolve `advertiserId`.

2. Pick the dimension:
   - **Which partners drove sales?** `getAdvertiserPublisherReport`
     (`GET /advertisers/{advertiserId}/reports/publisher`)
   - **Which campaigns?** `getAdvertiserCampaignReport`
     (`GET /advertisers/{advertiserId}/reports/campaign`)
   - **Which creatives?** `getAdvertiserCreativeReport`
     (`GET /advertisers/{advertiserId}/reports/creative`)

3. Window every call with `startDate`, `endDate` and an explicit `timezone`. Add `region` only
   when the question is country-scoped.

4. To go from an aggregate row down to the underlying sales, follow with
   `getAdvertiserTransactions` (`GET /advertisers/{advertiserId}/transactions/`) — but remember
   its window caps at **31 days**, while the report endpoints do not.

## Rules you must not break

- **20 calls per minute per user.** No `X-RateLimit-*` / `RateLimit-*` / `Retry-After` headers
  are returned, so budget client-side and back off with jitter on `429`.
- **The token is not scoped to one advertiser.** It reaches every account the human can reach.
  Never pull a second advertiser's numbers because it was convenient.
- **Do not present publisher-level rows outside the advertiser's own programme.** The publisher
  dimension names Awin partners; treat it as commercially sensitive to that programme.
- Currency is per programme — do not aggregate across programmes without converting.

## Errors

`403` = plan entitlement or missing Admin permission (permission changes take up to 10 minutes).
`429` = the shared user throttle. Full catalogue in `errors/awin-affiliate-problem-types.yml`.
