# Awin (awin-affiliate)

Awin is a global affiliate marketing network that connects advertisers (brands) with publishers (content creators, cashback, voucher, loyalty, and price-comparison partners) across thousands of programmes worldwide. Awin exposes a documented public REST API at `https://api.awin.com` that lets both publishers and advertisers pull data such as individual transactions and aggregated performance reports, inspect commission groups and programme details, list the accounts a user can access, and generate tracking links and offers.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/awin-affiliate/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/awin-affiliate/refs/heads/main/apis.yml)

## Access Model

Awin's API is publicly documented at [help.awin.com/apidocs](https://help.awin.com/apidocs/introduction-1), but calling it requires an Awin account and an access token, so it is not anonymously open.

- **Publishers** join the Awin network to earn commissions. Publisher signup is generally free to get started (some regions historically applied a small, refundable verification charge). Once approved, a publisher with Admin access can generate an API token.
- **Advertisers** pay to run programmes on Awin. Awin publishes advertiser plans (for example, Access at $49/month with a 3.5% per-transaction tracking fee, Accelerate from $99/month with 2.5%, and Advanced with custom pricing), plus the partner commission the advertiser itself sets. Advertisers with Admin access can also generate an API token.
- The **API itself carries no separate metered fee** — it is an access surface over the account you already hold. Cost sits in the network's commercial model (advertiser fees and publisher commissions), not in per-call API charges.

### Authentication

All APIs follow the OAuth 2.0 specification and require an access token passed as a Bearer token:

```
Authorization: Bearer <your_token>
```

Tokens are set at the **user level**: once created, a token grants access to data from every publisher or advertiser account the user is associated with. Tokens are generated at [ui.awin.com/awin-api](https://ui.awin.com/awin-api) (or the "API Credentials" user-menu item) and can be revoked there. The one exception is the **Create Transactions API**, which authenticates with an API key sent as `x-api-key` instead.

Only HTTPS is supported (there is no HTTP-to-HTTPS forwarding), responses default to JSON, and a throttle limits usage to **20 API calls per minute per user**.

## Tags

- Affiliate Marketing
- Advertising
- Publishers
- Advertisers
- Transactions
- Reporting
- Commissions
- Performance Marketing

## Timestamps

- **Created:** 2026-07-05
- **Modified:** 2026-07-05

## APIs

### Awin Accounts API

Return the list of Awin accounts the authenticated user can access. Because the access token is set at the user level, `GET /accounts` enumerates every publisher and advertiser account associated with the user, with an optional `type` filter, so integrations can discover account IDs before making per-account requests.

- **Human URL:** [https://help.awin.com/apidocs/returns-information-about-accounts-for-a-given-user](https://help.awin.com/apidocs/returns-information-about-accounts-for-a-given-user)
- **Base URL:** `https://api.awin.com`
- `GET /accounts`

### Awin Transactions API

Pull individual transactions (sales, leads, bonuses) for a publisher or advertiser over a date range of up to 31 days, filtered by timezone, date type (transaction, validation, amendment), and status (pending, approved, declined, deleted). Responses include click and order references, custom parameters, commission, conversion lapse time, and optional basket products.

- **Human URL:** [https://help.awin.com/apidocs/returns-a-list-of-transactions-for-a-given-publisher](https://help.awin.com/apidocs/returns-a-list-of-transactions-for-a-given-publisher)
- **Base URL:** `https://api.awin.com`
- `GET /publishers/{publisherId}/transactions/`
- `GET /advertisers/{advertiserId}/transactions/`

### Awin Reports API

Retrieve aggregated performance reports over a date range. Publishers aggregate by advertiser, creative, or campaign; advertisers aggregate by publisher, creative, or campaign. Each report breaks impressions, clicks, and transaction counts, values, and commission down by status (pending, confirmed, bonus, declined) with totals, region, and currency.

- **Human URL:** [https://help.awin.com/apidocs/get-advertiser-performance-report](https://help.awin.com/apidocs/get-advertiser-performance-report)
- **Base URL:** `https://api.awin.com`
- `GET /publishers/{publisherId}/reports/advertiser`
- `GET /publishers/{publisherId}/reports/creative`
- `GET /publishers/{publisherId}/reports/campaign`
- `GET /advertisers/{advertiserId}/reports/publisher`
- `GET /advertisers/{advertiserId}/reports/creative`
- `GET /advertisers/{advertiserId}/reports/campaign`

### Awin Commission Groups API

Request all commission groups of a programme along with the commission values a publisher earns. Returns each group's id, name, code, type (fix or percentage), amount or percentage, currency, effective rate window, and any applicable conditions, with an optional `effectiveDate` for historical rates.

- **Human URL:** [https://help.awin.com/apidocs/gets-an-array-of-commission-groups-for-an-advertiser](https://help.awin.com/apidocs/gets-an-array-of-commission-groups-for-an-advertiser)
- **Base URL:** `https://api.awin.com`
- `GET /publishers/{publisherId}/commissiongroups`

### Awin Programmes API

Discover the advertiser programmes a publisher works with and their details. List programmes filtered by relationship (joined, pending, suspended, rejected, notjoined, any) and optionally by country, then pull per-programme detail including description, membership status, valid domains, KPIs (EPC, conversion rate, approval percentage, average payment time, Awin index) and the commission range across commission groups.

- **Human URL:** [https://help.awin.com/apidocs/get-program-information-details-for-publisher](https://help.awin.com/apidocs/get-program-information-details-for-publisher)
- **Base URL:** `https://api.awin.com`
- `GET /publishers/{publisherId}/programmes`
- `GET /publishers/{publisherId}/programmedetails`

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/awin)
- [Website](https://www.awin.com)
- [Documentation](https://help.awin.com/apidocs/introduction-1)
- [Authentication](https://help.awin.com/apidocs/api-authentication)
- [Plans](plans/awin-affiliate-plans-pricing.yml)
- [Rate Limits](rate-limits/awin-affiliate-rate-limits.yml)
- [Fin Ops](finops/awin-affiliate-finops.yml)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
