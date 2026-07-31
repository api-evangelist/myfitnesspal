---
name: Subscribe to a user's data-change webhooks and process notifications
description: Register a MyFitnessPal webhook subscription and correctly process the batched change notifications.
api: https://api.myfitnesspal.com/v2
operations:
  - POST /v2/subscription
  - GET /v2/subscription
  - DELETE /v2/subscription
scopes:
  - subscriptions
generated: '2026-07-20'
method: generated
---

# Subscribe to user updates via webhooks

Use this to be notified when a consenting user's diary, measurements, or profile change, instead of polling.

## Prerequisites
- An OAuth 2.0 access token granted the **`subscriptions`** scope.
- Send the **`mfp-client-id`** header; set `Content-Type: application/json`.
- A public HTTPS callback endpoint you control.

## Steps
1. **Create a subscription** — `POST /v2/subscription` with your callback URL in the request body. Returns `201 Created`.
2. **Verify / list subscriptions** — `GET /v2/subscription`.
3. **Receive notifications** — MyFitnessPal `POST`s a JSON **array** of notifications to your callback. Each item has `user_id`, `item_type` (`measurements`, `diary_exercise`, `diary_meal`, or `user_info`), and `item_url` (the v2 URL to fetch the changed resource). A single request may batch **thousands** of items across many users.
4. **Acknowledge fast, process async** — respond `202 Accepted` immediately and process the batch asynchronously; then `GET` each `item_url` (with the appropriate scope) to fetch the change.
5. **Unsubscribe** — `DELETE /v2/subscription` when no longer needed.

## Rules
- Never do heavy work before acknowledging; close the connection quickly to avoid redelivery/timeouts.
- Fetching `item_url` for `diary_*` requires the `diary` scope; for `measurements` requires the `measurements` scope; for `user_info` requires `diary`.
- Missing/invalid required headers return `missing-header` / `malformed-header`; an expired token returns `401`.
