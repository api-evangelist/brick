---
name: brick-model-ingest-and-query
description: Upload a Brick model as Turtle to a Brick Example Server and query the entity graph by Brick relationship.
api: Brick Example Server
generated: '2026-09-04'
method: generated
source: openapi/brick-brick-server-openapi.yml
operations:
  - upload_brickapi_v1_entities_upload_post
  - post_brickapi_v1_entities_list_post
  - get_entity_by_id_brickapi_v1_entities__get
  - gen_token_brickapi_v1_auth_app_tokens_post
---

# Ingest a Brick model and query it by relationship

The Brick Example Server is **self-hosted software**. There is no hosted base URL — substitute the
host your operator runs. Every path below is exactly as published in the contract, under
`/brickapi/v1/`.

## Before you start

- Validate the model OFFLINE first. `pip install brickschema` then
  `brick_validate myBuilding.ttl` checks it against the Brick SHACL shapes. The server publishes no
  dry-run or validate-only mode, so this is your only rehearsal.
- Get a bearer token. `POST /brickapi/v1/auth/app_tokens` (`gen_token_brickapi_v1_auth_app_tokens_post`)
  with optional `app_name` and `token_lifetime` (seconds) returns `{token, name, exp}`. Send it as
  `Authorization: Bearer <token>`. All three operations below require it.

## 1. Upload the model

`POST /brickapi/v1/entities/upload` — `upload_brickapi_v1_entities_upload_post`

- `multipart/form-data` with a `file` field carrying the Turtle document.
- Optional query parameter `named_graph` — "The name of the graph. This is similar to a database
  name". Use it to keep buildings separate.
- **There is no undo.** No entity-delete, graph-version or rollback operation exists in the contract.
  Upload into a fresh `named_graph` when you are unsure, rather than into a live one.
- On a malformed request you get `422` with `HTTPValidationError` — read `detail[].loc` for the
  offending field.

## 2. Find entities by relationship

`POST /brickapi/v1/entities/list` — `post_brickapi_v1_entities_list_post`

The `ListEntityParams` body is the Brick ontology's core relationship set. Each field takes a list of
object URIs for that predicate:

`hasPoint`, `isPointOf`, `hasPart`, `isPartOf`, `hasLocation`, `isLocationOf`, `feeds`, `isFedBy`

Example intent: "every point on AHU-1" → send `isPointOf: ["<AHU-1 IRI>"]`. "Everything AHU-1 feeds"
→ `isFedBy: ["<AHU-1 IRI>"]`.

There is **no pagination** on this operation. The response is the whole matching set — expect large
bodies on a real building and filter narrowly.

## 3. Read one entity

`GET /brickapi/v1/entities/?entity_id=<uri>` — `get_entity_by_id_brickapi_v1_entities__get`

Returns `Entity`: `entity_id` (the identifier, "often a URI"), `type` ("often a Brick Class"),
`name`, and `relationships` — the entity's own edges in the graph.

## Conventions that apply throughout

- Auth: bearer JWT. No scopes exist; a token is all-or-nothing.
- Errors: only `422` is documented. There is no declared `401`, `403`, `404`, `429` or `5xx` — treat
  any other status as undocumented and do not assume it is retryable.
- No idempotency key, no request-id header, no rate-limit headers.
