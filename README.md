# Awin (awin-affiliate)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
