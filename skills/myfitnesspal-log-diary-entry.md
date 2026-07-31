---
name: Log a food or exercise entry to a user's diary
description: Create, read, and update entries in a MyFitnessPal user's food and exercise diary via the v2 API.
api: https://api.myfitnesspal.com/v2
operations:
  - POST /v2/diary
  - GET /v2/diary
  - PATCH /v2/diary
scopes:
  - diary
generated: '2026-07-20'
method: generated
---

# Log a diary entry

Use this to add a meal (`diary_meal`) or exercise (`diary_exercise`) entry to a consenting MyFitnessPal user's diary.

## Prerequisites
- An OAuth 2.0 access token granted the **`diary`** scope (authorization-code grant).
- Send the partner **`mfp-client-id`** header on every request; use **`mfp-user-id`** to target the user.
- Set `Content-Type: application/json`.

## Steps
1. **Create the entry** — `POST /v2/diary` with a JSON body describing the entry and its `item_type` (`diary_meal` or `diary_exercise`). A `diary_meal` carries nutritional contents; a `diary_exercise` references an exercise. On success you receive `201 Created`.
2. **Read entries back** — `GET /v2/diary` (optionally filtered, e.g. `?types=diary_meal&entry_date=YYYY-MM-DD`). Use the `fields` query parameter to limit returned fields.
3. **Correct an entry** — `PATCH /v2/diary` with only the changed fields; a successful update returns `204 No Content`.

## Rules
- Do not resubmit a create after a network error without checking with `GET /v2/diary`; the API has no idempotency-key mechanism and a client-assigned duplicate id returns `duplicate-id` / `uniqueness-violation`.
- Validation failures return `422 Unprocessable Entity` with a `failed-validation` error; read `error_description` (localized) and `error_details`.
- Back off on `429 Too Many Requests`.
- Only `diary_meal` and `diary_exercise` entry types are supported; an unknown type returns `unsupported-diary-type`.
