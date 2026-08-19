---
name: awin-publisher-performance-reporting
description: Build a publisher's Awin performance picture across advertiser, campaign and creative dimensions, and know why the three reports will not tie out to each other.
api: openapi/awin-affiliate-reports-api-openapi.yml
operations:
  - getAccounts
  - getPublisherAdvertiserReport
  - getPublisherCampaignReport
  - getPublisherCreativeReport
generated: '2026-08-13'
method: generated
source: openapi/_original/awin-affiliate-openapi.yml + https://help.awin.com/apidocs/reports-1
---

# Report Awin publisher performance

Use this to answer "how did this publisher do last month, and where did it come from?"
without pulling every individual transaction.

## Before you call

Bearer token from <https://ui.awin.com/awin-api>, sent as `Authorization: Bearer <token>`.
Reports are aggregates — cheaper than transaction pulls and the right first call when the
question is about totals rather than individual sales.

## Steps

1. `getAccounts` (`GET /accounts`, `type=publisher`) → resolve `publisherId`.

2. Pick the dimension that answers the question. All three take the same
   `startDate` / `endDate` / `timezone` window:
   - **Which brands earned?** `getPublisherAdvertiserReport`
     (`GET /publishers/{publisherId}/reports/advertiser`)
   - **Which campaigns earned?** `getPublisherCampaignReport`
     (`GET /publishers/{publisherId}/reports/campaign`)
   - **Which creatives earned?** `getPublisherCreativeReport`
     (`GET /publishers/{publisherId}/reports/creative`)

3. Send `region` only when the user asked for a country cut, and always send `timezone` — the
   day boundary is what decides which month a sale lands in.

4. Use `dateType` deliberately. Transaction date answers "when did the sale happen"; validation
   date answers "what will be paid in this statement". These give different numbers for the same
   window and the difference is not a bug.

## Rules you must not break

- **20 calls per minute per user**, and no rate-limit response headers exist. Count your own calls.
- **Do not add the three reports together.** They are three projections of the same underlying
  transactions along different dimensions. Creative-level rows will not exist for traffic that
  did not go through a creative, so the creative report legitimately totals less than the
  advertiser report.
- **Currency.** Rows carry a currency; do not aggregate across programmes without converting.
- If the user needs sale-by-sale detail, switch to the transaction reconciliation skill instead
  of trying to reconstruct it from report rows.

## Errors

`403` usually means the user lacks Admin permission on that publisher account (up to 10 minutes
to propagate after a permission change). Full catalogue in
`errors/awin-affiliate-problem-types.yml`.
