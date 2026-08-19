---
name: Run a local discovery audit with the Zoca public API
description: Use Zoca's unauthenticated lead-magnet endpoints to profile a local business — keyword demand, Google Business Profile competitors, place metrics and a website strategy grade — without an account.
api: openapi/zoca-public-openapi.yml
base_url: https://public.zoca.com
operations:
  - "GET /lead-magnet/place/details"
  - "GET /lead-magnet/place/details/keywords"
  - "GET /lead-magnet/place/details/competitor-table"
  - "GET /lead-magnet/keyword/demand"
  - "POST /lead-magnet/keyword-intelligence"
  - "GET /lead-magnet/gbp/competitor"
  - "GET /lead-magnet/website-strategy"
  - "GET /lead-magnet/get-insights"
  - "POST /lead-magnet/export"
generated: '2026-08-13'
method: generated
source: openapi/zoca-public-openapi.yml
---

# Run a local discovery audit with the Zoca public API

This is the **only Zoca surface reachable without a Zoca account.** `https://public.zoca.com` backs the free
self-serve tools on zoca.com (Local Business Demand Tracker, GBP Optimizer, Website Grader). The contract
declares the `access-token` bearer scheme, but the lead-magnet operations answer unauthenticated — that is what
the free tools on the marketing site are calling.

Operations are identified by METHOD + path. `operationId` is `t_value` on every operation in this spec.

## Steps

1. **Resolve the business.** `GET /lead-magnet/place/details` returns the place record. Everything downstream
   keys off it.
2. **Pull its keyword surface.** `GET /lead-magnet/place/details/keywords`, then
   `GET /lead-magnet/place/details/competitor-table` for the head-to-head table.
3. **Size the demand.** `GET /lead-magnet/keyword/demand` for a single keyword;
   `POST /lead-magnet/keyword-intelligence` for the batch/analysis form.
4. **Profile the Google Business Profile competition.** `GET /lead-magnet/gbp/competitor`.
5. **Grade the website.** `GET /lead-magnet/website-strategy`. Note there is a matching
   `GET /lead-magnet/website-strategy/callback` — that is an **inbound receiver** for Zoca's own async grader,
   not something you call.
6. **Roll it up.** `GET /lead-magnet/get-insights`, then `POST /lead-magnet/export` to emit the report.

## Rules

- **Eight of the pricing operations on this host declare a bare `429` "Too many requests"** with no published
  limit, no window and no `Retry-After` header. Rate yourself conservatively and back off on 429 — you have no
  budget signal. See `rate-limits/zoca-rate-limits.yml`.
- This is a **public marketing surface, not a product API**. Zoca publishes no developer portal, no terms of
  API use, and no plan that grants access to it. Treat it as unsupported and subject to change without notice:
  there is no changelog, no versioning policy and no deprecation policy
  (`lifecycle/zoca-lifecycle.yml`).
- Response shapes are undiscoverable from the contract — `components.schemas` holds two anonymous minified
  schemas. Inspect at runtime.
