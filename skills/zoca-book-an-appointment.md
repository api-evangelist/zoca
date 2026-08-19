---
name: Book an appointment through the Zoca FrontDesk widget
description: Take a client from "I want a haircut on Friday" to a confirmed Zoca appointment using the FrontDesk booking-widget surface — catalogue, availability, price quote, slot hold, and booking.
api: openapi/zoca-tasks-openapi.yml
base_url: https://tasks.zoca.ai
operations:
  - "GET /tasks/api/v1/frontdesk/widget/{entityId}/config"
  - "GET /tasks/api/v1/frontdesk/widget/{entityId}/services"
  - "GET /tasks/api/v1/frontdesk/widget/{entityId}/staff"
  - "GET /tasks/api/v1/frontdesk/widget/{entityId}/addons"
  - "GET /tasks/api/v1/frontdesk/widget/{entityId}/availability"
  - "GET /tasks/api/v1/frontdesk/widget/{entityId}/slots"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/identify"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/price"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/verify-promo"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/hold"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/client"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/card-status"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/book"
generated: '2026-08-13'
method: generated
source: openapi/zoca-tasks-openapi.yml
---

# Book an appointment through the Zoca FrontDesk widget

**Operations are identified by METHOD + path, not by `operationId`.** Zoca's build emitted the literal string
`t_value` as the `operationId` on 2,621 of its 2,624 published operations, so `operationId` is not a usable
handle. Every step below was verified to exist verbatim in `openapi/zoca-tasks-openapi.yml`.

## Before you start

- Base URL: `https://tasks.zoca.ai`
- Auth: `Authorization: Bearer <JWT>` (scheme `access-token`). See `authentication/zoca-authentication.yml`.
- `{entityId}` is the Zoca **entity** — one business location. It scopes every operation in this flow.
  See `data-model/zoca-data-model.yml`.
- Zoca publishes no sandbox and no test data. Do not exercise write steps against a real location.

## Steps

1. **Load the widget contract.** `GET /tasks/api/v1/frontdesk/widget/{entityId}/config` — first page of services,
   categories and booking preferences. Read the preferences before anything else; they decide whether a card on
   file is required and whether multi-service carts are allowed.
2. **Resolve what the client wants.** `GET /tasks/api/v1/frontdesk/widget/{entityId}/services` (paginated and
   searchable — use `page`/`limit`), then `GET .../staff` for providers and `GET .../addons` for suggestable
   extras and staff surcharges.
3. **Find a date.** `GET /tasks/api/v1/frontdesk/widget/{entityId}/availability` returns dates with openings,
   grouped by date, for a service. Only then `GET .../slots` for the concrete times on the chosen date.
4. **Quote, do not charge.** `POST /tasks/api/v1/frontdesk/widget/{entityId}/identify` resolves the customer's
   pricing tier; `POST .../price` returns a **display-only** quote and never charges. If the client has a code,
   `POST .../verify-promo` verifies it without redeeming it. Never present a price you did not get from `/price`.
5. **Hold the slot before you collect anything.** `POST /tasks/api/v1/frontdesk/widget/{entityId}/hold` reserves
   the slot with a timer. Do this before asking the client for name, phone or card — a hold that expires is
   cheap, a booking that fails after you took a card is not.
6. **Attach the client.** `POST /tasks/api/v1/frontdesk/widget/{entityId}/client` finds-or-creates the platform
   client. Then `POST .../card-status` to learn whether this entity requires a card on file and whether this
   client already has one.
7. **Book.** `POST /tasks/api/v1/frontdesk/widget/{entityId}/book` creates the appointment.

## Rules

- **Retries are unsafe.** `POST .../book` is **not** documented as idempotent and declares no `Idempotency-Key`
  parameter. On a timeout or a 5xx, do **not** blind-retry — re-read availability and confirm with the client.
  Only 27 of Zoca's 2,624 operations are documented as idempotent, and `book` is not one of them.
  See `conventions/zoca-conventions.yml`.
- **Errors** come back as `{"statusCode":…,"message":…,"error":…}` — the NestJS envelope, not RFC 9457 problem
  details. There is no error-code registry; branch on the HTTP status, not on the message string.
  See `errors/zoca-problem-types.yml`.
- **Rate limits are invisible.** No `RateLimit-*` or `Retry-After` header is declared on any operation. You will
  get a bare `429` with no budget signal. Back off exponentially. See `rate-limits/zoca-rate-limits.yml`.
- Response bodies are **not typed** in the published contract (`components.schemas` is minified to `e`/`t`/
  `Object`), so field names must be discovered at runtime. Do not assume a shape.
