---
name: Plan, generate and schedule content in the Zoca content queue
description: Drive Zoca's AI content pipeline for one location — build a plan, generate items, inspect the calendar and stats, and reschedule or publish a queued item.
api: openapi/zoca-tasks-openapi.yml
base_url: https://tasks.zoca.ai
operations:
  - "GET /tasks/api/v1/content-queue/{entityId}/plan"
  - "POST /tasks/api/v1/content-queue/{entityId}/plan/generate"
  - "GET /tasks/api/v1/content-queue/{entityId}/items"
  - "POST /tasks/api/v1/content-queue/{entityId}/items"
  - "POST /tasks/api/v1/content-queue/{entityId}/items/{itemId}/generate"
  - "POST /tasks/api/v1/content-queue/{entityId}/items/{itemId}/reschedule"
  - "PATCH /tasks/api/v1/content-queue/{entityId}/items/{itemId}/post"
  - "GET /tasks/api/v1/content-queue/{entityId}/calendar"
  - "GET /tasks/api/v1/content-queue/{entityId}/stats"
generated: '2026-08-13'
method: generated
source: openapi/zoca-tasks-openapi.yml
---

# Plan, generate and schedule content in the Zoca content queue

The content queue is the engine behind Zoca's Social Agent. Operations are identified by METHOD + path;
`operationId` is `t_value` throughout.

## Steps

1. **Read the plan before generating anything.** `GET /tasks/api/v1/content-queue/{entityId}/plan` — and
   `GET .../plan/summary` and `GET .../plan/services` for the breakdown. Generating on top of an existing plan
   is how you get duplicate posts.
2. **Build the plan** with `POST /tasks/api/v1/content-queue/{entityId}/plan/generate` if there isn't one.
3. **Work the items.** `GET .../items` lists them (paginated: `page`, `limit`). `POST .../items` adds one.
   `POST .../items/{itemId}/generate` fills in a single item's content.
4. **Move things.** `POST .../items/{itemId}/reschedule` for one item.
5. **Publish.** `PATCH .../items/{itemId}/post`.
6. **Check your work.** `GET .../calendar` for the schedule view, `GET .../stats` for the rollup.

## Rules

- **The bulk and force operations are the dangerous ones.** The same contract exposes
  `POST .../items/bulk-force-publish`, `POST .../items/{itemId}/force-publish`,
  `POST .../items/bulk-delete`, `POST .../generate-all` and
  `POST .../items/force-all-category-pages`. These publish to a real salon's live social accounts and website
  and are **not** documented as idempotent or reversible. Never call a `force-*`, `bulk-*` or `*-all` operation
  from an autonomous loop; require an explicit human confirmation for each one.
- `POST .../plan/generate` and `POST .../items/{itemId}/generate` are AI generation calls. Budget for latency
  and for a `429`; there is no published rate limit and no `Retry-After`.
- Errors are the NestJS `{statusCode, message, error}` envelope. See `errors/zoca-problem-types.yml`.
