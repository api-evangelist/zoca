---
name: Capture a booking enquiry and hand it to the Zoca Win agent
description: Turn an inbound enquiry into a tracked Zoca lead and enqueue the Win conversion agent to chase it, using the widget lead endpoint, the funnel reporter, and the win-conversion trigger.
api: openapi/zoca-tasks-openapi.yml
base_url: https://tasks.zoca.ai
operations:
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/funnel"
  - "POST /tasks/api/v1/frontdesk/widget/{entityId}/lead"
  - "POST /tasks/api/v1/win-conversion/triggers/booking-enquiry"
  - "GET /booking-enquiries"
generated: '2026-08-13'
method: generated
source: openapi/zoca-tasks-openapi.yml, openapi/zoca-platform-openapi.yml
---

# Capture a booking enquiry and hand it to the Zoca Win agent

Zoca's "Win Agent" is the product that converts enquiries into confirmed appointments. This is the operation
sequence behind it. Operations are identified by METHOD + path — `operationId` is `t_value` throughout and
cannot be used.

## Steps

1. **Report where the client got to.** `POST /tasks/api/v1/frontdesk/widget/{entityId}/funnel` records a funnel
   step (service selected, slot selected). Call it as the client moves, not at the end — an abandoned funnel is
   the signal the Win agent acts on.
2. **Create the enquiry.** `POST /tasks/api/v1/frontdesk/widget/{entityId}/lead` creates a booking inquiry and
   engages lead follow-up. This is the correct endpoint when the client did **not** complete a booking.
3. **Enqueue the Win conversion agent.** `POST /tasks/api/v1/win-conversion/triggers/booking-enquiry` validates
   the payload and enqueues the job. It returns **202 Accepted**, not 200 — treat it as accepted-for-processing,
   never as converted.
4. **Read back.** `GET /booking-enquiries` on the platform service (`https://api.zoca.ai`) lists enquiries for
   reconciliation.

## Rules

- **This is the one operation in Zoca's whole surface with a documented idempotency key.**
  `POST /tasks/api/v1/win-conversion/triggers/booking-enquiry` is documented as: *"Idempotent — repeat calls with
  the same idempotency key return `alreadyEnqueued=true`."* Send a stable key per enquiry and treat
  `alreadyEnqueued=true` as success, not as a duplicate error. The key is **not** declared as a parameter in the
  spec, so read it from the request body shape at runtime.
- Steps 1 and 2 are **not** idempotent. Calling `/lead` twice creates two enquiries and can trigger two
  follow-up sequences at a real client. Guard with your own dedupe.
- Two different base hosts are in play here: `https://tasks.zoca.ai` for steps 1–3 and `https://api.zoca.ai`
  for step 4. They are separate services on separate release versions (3.20.9 and 3.20.10).
