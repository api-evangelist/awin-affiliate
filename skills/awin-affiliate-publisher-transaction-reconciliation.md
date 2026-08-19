---
name: awin-publisher-transaction-reconciliation
description: Pull a publisher's Awin transactions for a period and reconcile them against the programme's commission groups, respecting the 31-day window cap and the 20-call-per-minute user throttle.
api: openapi/awin-affiliate-transactions-api-openapi.yml
operations:
  - getAccounts
  - getPublisherTransactions
  - getCommissionGroups
generated: '2026-08-13'
method: generated
source: openapi/_original/awin-affiliate-openapi.yml + https://help.awin.com/apidocs/introduction-1
---

# Reconcile Awin publisher transactions

Use this when a publisher needs the individual transactions behind a period's earnings —
for cashback attribution, invoice checking, or feeding a data warehouse.

## Before you call

- **Token.** A single user-level token, minted by a human at <https://ui.awin.com/awin-api>.
  Send it as `Authorization: Bearer <token>`. Some endpoints also accept it as an
  `accessToken` query parameter — prefer the header; the query form leaks into logs.
- **There is no sandbox.** Every call hits production data. See `sandbox/awin-affiliate-sandbox.yml`.
- **The token is not scoped.** It reaches every account the human can reach. Never widen a
  request beyond the account the user actually asked about.

## Steps

1. **Resolve the accounts.** Call `getAccounts` (`GET /accounts`). Filter with `type=publisher`
   to get only publisher accounts. Take the `id` of the account the user named — do not guess
   when more than one comes back; ask.

2. **Read the commission groups** for each advertiser programme you will reconcile against, with
   `getCommissionGroups` (`GET /publishers/{publisherId}/commissiongroups`). Key them by `code` —
   that is the value transactions reference, not the numeric `id`.

3. **Pull the transactions** with `getPublisherTransactions`
   (`GET /publishers/{publisherId}/transactions/`).
   - `startDate` / `endDate` must span **31 days or fewer**. For a longer period, split into
     consecutive ≤31-day chunks and call once per chunk.
   - Always send `timezone` explicitly — it is part of transaction identity in Awin, not a
     display preference.
   - Use `dateType` to choose whether the window filters on transaction date or validation date;
     reconciling against a payment statement usually means validation date.
   - Set `showBasketProducts` only when you actually need product lines; it inflates the payload.

4. **Reconcile.** For each transaction, sum `commissionGroups[].amount` and confirm it matches
   `transactionAmount`. Group the commission by `commissionGroups[].code` to compare against the
   published rates from step 2.

## Rules you must not break

- **Throttle: 20 calls per minute per USER**, shared across every account that token reaches.
  A parallel fan-out over many programmes will 429. Serialize, and back off exponentially with
  jitter on `429` — Awin returns **no** rate-limit headers, so you cannot read your remaining
  budget; you must count your own calls.
- **Never sum `transactionAmount` across programmes without normalising currency.** A transaction
  can carry `transactionCurrency` (the programme's) and `trackedCurrency` (what the shopper paid
  in) with different amounts.
- **No idempotency contract exists** anywhere in this API. These are reads, so retry is safe —
  but do not carry that assumption into any Awin write surface.
- Report `403` as an entitlement problem, not a bug: publisher admin permission changes take up
  to 10 minutes to propagate to a token.

## Errors

See `errors/awin-affiliate-problem-types.yml`. The bodies are
`{"error": "...", "description": "..."}` — there is no RFC 9457 problem+json and no error code
beyond the HTTP status.
