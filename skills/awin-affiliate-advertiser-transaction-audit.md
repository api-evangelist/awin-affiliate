---
name: awin-advertiser-transaction-audit
description: Pull an advertiser's individual Awin transactions for audit or dispute, and understand the validation states and identity rules before touching the batch validation surface.
api: openapi/awin-affiliate-transactions-api-openapi.yml
operations:
  - getAccounts
  - getAdvertiserTransactions
generated: '2026-08-13'
method: generated
source: openapi/_original/awin-affiliate-openapi.yml + https://help.awin.com/apidocs/approve-decline-amend-batch-transactions-for-a-given-advertiser
---

# Audit Awin advertiser transactions

Use this when an advertiser needs the sale-by-sale record — validating commissions, chasing a
disputed order, or matching Awin against their own order system.

## Before you call

- Bearer token from <https://ui.awin.com/awin-api>, `Authorization: Bearer <token>`.
- **Advertiser API access requires the Accelerate or Advanced plan.** A `403` here is usually
  commercial.
- **No sandbox exists.** Reads are safe; anything you plan afterwards on the validation surface
  runs against production money.

## Steps

1. `getAccounts` (`GET /accounts`, `type=advertiser`) → resolve `advertiserId`.

2. `getAdvertiserTransactions` (`GET /advertisers/{advertiserId}/transactions/`):
   - `startDate` / `endDate` must span **31 days or fewer** — chunk longer audits.
   - Send `timezone` explicitly. In Awin, `(orderRef, transactionDate, timezone)` is a valid
     **primary key** for a transaction, so timezone is identity, not formatting.
   - `status` filters by validation state (pending / approved / declined). Auditing "what will
     I be billed" and "what did I already pay" are different `status` calls.
   - `showBasketProducts` adds product lines — only when the audit needs them.

3. Match against the advertiser's own orders on `orderRef`, falling back to `transactionId`.

## If the audit leads to a correction

Awin's Batch Validation endpoint (`POST /advertisers/{advertiserId}/transactions/batch`) approves,
declines, amends sale amounts and amends tracking parameters — up to 40,000 transaction objects in
one request. It is **not** modelled in this repo's OpenAPI and is **not** part of this skill.
Before you go near it, know:

- **There is no idempotency key.** Resubmitting a batch is not a safe retry, and Awin documents
  no duplicate-suppression behaviour.
- **`304` is not an error and not a cache signal** — it means the transaction was already in the
  requested state (declining an already-declined sale).
- **`422` has documented business causes**: CPO advertisers, transactions whose sale amount or
  commission was previously amended, shared transactions, subnetwork-publisher transactions, and
  anything dated before June 2023.
- On an amend, `transactionParts[]` amounts **must** sum to `saleAmount`, or Awin silently
  adjusts the `DEFAULT` commission group to balance the difference.
- Always confirm with a human before executing a batch that changes commission. This is money.

## Rules you must not break

- **20 calls per minute per user**, shared across every account the token reaches, with no
  rate-limit headers to read.
- Currency: `transactionCurrency` is the programme's; `trackedCurrency` / `trackedAmount` is what
  the shopper actually paid when the two differ.
- Never widen the date window past 31 days to "save calls" — the request will fail.

## Errors

Full catalogue in `errors/awin-affiliate-problem-types.yml`.
