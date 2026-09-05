---
name: brick-actuation-safety
description: Actuate a physical control point through a Brick Example Server, and understand why this write has no undo.
api: Brick Example Server
generated: '2026-09-04'
method: generated
source: openapi/brick-brick-server-openapi.yml
operations:
  - post_brickapi_v1_actuation__post
  - get_entity_by_id_brickapi_v1_entities__get
  - get_brickapi_v1_data_timeseries_get
---

# Actuate a Brick point — and what the contract does not give you

`POST /brickapi/v1/actuation/` — `post_brickapi_v1_actuation__post`

- Query parameter `entity_id` (required): "The identifier of an entity. Often a URI."
- JSON body `ActuationRequest`: `{ "value": <number> }` — "A value to set the target entity."
- Bearer JWT required.

This operation writes to physical building equipment. Read the rest of this page before calling it.

## What the contract does NOT provide

Verified against `openapi/brick-brick-server-openapi.yml` on 2026-09-04:

- **No reversal.** There is no cancel, revert, undo or restore operation anywhere in the contract.
- **No previous value.** The operation does not return what the point was set to before, and no
  read-back operation exists for a control point's current setpoint.
- **No idempotency key.** A retry after a timeout re-actuates. There is no way to make the second call
  a no-op.
- **No dry run.** No preview or validate-only parameter exists.
- **No documented failure modes.** Only `422` is declared; there is no `401`, `403`, `404`, `409` or
  `5xx` in the contract, and some operations report failure inside a `200` body via `IsSuccess`
  (`is_success`, `reason`) instead of a status code.

## Therefore, before you actuate

1. Confirm the target with `GET /brickapi/v1/entities/?entity_id=<uri>` and check `type` is a Brick
   class you meant to write to. `brick:Command` and setpoint classes are writable targets; a sensor
   point is not.
2. Capture the current state yourself with
   `GET /brickapi/v1/data/timeseries?entity_id=<uri>&start_time=…&end_time=…`. **This is your only
   rollback** — a second actuation back to the captured value.
3. Send exactly one actuation. On timeout, do not retry blind: re-read the timeseries and decide.
4. Record the prior value and the new value on your side. The server records neither for you.

## Conventions

- Auth: bearer JWT, no scopes — a token that can read can also actuate.
- Errors: `422` with `HTTPValidationError` (`detail[].loc`, `detail[].msg`, `detail[].type`).
- Rate limits: none published; the operator sets them.
