---
name: Authenticate and run a QuasarDB query
description: Log in to the QuasarDB REST API, obtain a JWT, and run a SQL query against a time-series cluster.
api: openapi/quasar-rest-openapi-original.json
operations: [login, post-query, get-tags, get-table-csv]
---

# Authenticate and run a QuasarDB query

Use the QuasarDB REST API (default `https://<host>:40443/api`, or `http://<host>:40080/api` for insecure) to query a cluster.

## Steps

1. **Authenticate.** Call `login` — `POST /api/login` with body `{"username": "<user>", "secret_key": "<base64 user private key>"}`. On success (200) you get a `Token` object `{"token": "<jwt>"}`. The JWT is valid for 12 hours. A 401 means bad credentials.
2. **Set the auth header.** Send `Authorization: Bearer <jwt>` on every subsequent request (or pass it as the `token` query-string parameter if you cannot set headers).
3. **(Optional) discover tables/tags.** Call `get-tags` — `GET /api/tags` — to list tags used to organize tables.
4. **Run the query.** Call `post-query` — `POST /api/query` with body `{"query": "SELECT ... FROM <table> IN RANGE(...)"}`. The response is a `QueryResult` with `tables[] -> columns[] {name, type, data[]}` (columnar JSON). Express paging/limits inside the SQL (`LIMIT`), not via API parameters.
5. **(Optional) bulk export.** For a whole table, call `get-table-csv` — `GET /api/tables/{name}.csv` — to stream CSV.

## Rules

- Errors return `{"message": "<text>"}` (not RFC 9457). Handle 400 (bad query), 401 (auth), 500 (cluster error) — see `errors/quasar-problem-types.yml`.
- There is no idempotency key; do not blind-retry writes issued through `SELECT ... INSERT`-style statements.
- RBAC privileges (select/insert/update/…) gate what the authenticated user may run.
