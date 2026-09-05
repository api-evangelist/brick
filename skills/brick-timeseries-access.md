---
name: brick-timeseries-access
description: Read, write and delete timeseries data for a Brick point on a Brick Example Server, including the raw SQL and SPARQL escape hatches.
api: Brick Example Server
generated: '2026-09-04'
method: generated
source: openapi/brick-brick-server-openapi.yml
operations:
  - get_brickapi_v1_data_timeseries_get
  - post_brickapi_v1_data_timeseries_post
  - delete_brickapi_v1_data_timeseries_delete
  - post_brickapi_v1_rawqueries_timeseries_post
  - post_brickapi_v1_rawqueries_sparql_post
---

# Work with timeseries data behind a Brick model

Self-hosted server; substitute your operator's host. Bearer JWT required on every operation here.

## Read a point's history

`GET /brickapi/v1/data/timeseries` — `get_brickapi_v1_data_timeseries_get`

| Parameter | In | Required | Notes |
|---|---|---|---|
| `entity_id` | query | yes | The Brick point IRI |
| `start_time` | query | no | "Starting time of the data in UNIX timestamp in seconds (float)" |
| `end_time` | query | no | "Ending time of the data in UNIX timestamp in seconds (float)" |
| `value_types` | query | no | Array; "The type of value" — numbers and text |

Response is `TimeseriesData`: `data` is an array of value tuples, and `columns` "explain how to
interpret the values in the data". Read `columns` first — do not assume tuple order.

**Always send a bounded `start_time`/`end_time`.** Both are optional and there is no pagination and no
row limit in the contract, so an unbounded read on a busy point returns everything.

## Write rows

`POST /brickapi/v1/data/timeseries` — `post_brickapi_v1_data_timeseries_post`

`application/json` body is a `TimeseriesData` object — "A table of data where each row represents a
value tuple." No idempotency key exists, so a retried write can duplicate rows. Reconcile with a read
before retrying rather than firing again.

## Delete rows

`DELETE /brickapi/v1/data/timeseries` — `delete_brickapi_v1_data_timeseries_delete`

Takes `entity_id` (required) plus optional `start_time` / `end_time`. **Omitting the time bounds is
the dangerous case** — the contract states no default window, so treat an unbounded delete as
deleting everything for that entity. There is no restore operation and no stated retention window.

## Escape hatches

- `POST /brickapi/v1/rawqueries/timeseries` — `application/sql` body, "A raw SQL query for timeseries
  data. The table consist of the columns as in `value_types`."
- `POST /brickapi/v1/rawqueries/sparql` — `application/sparql-query` body, "Raw SPARQL for Brick
  metadata." Its own description warns it "May not be exposed in the production deployment", so probe
  for availability before depending on it.

Both are unbounded query surfaces on someone's building data. Prefer the typed operations above.
