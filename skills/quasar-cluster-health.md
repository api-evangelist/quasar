---
name: Monitor QuasarDB cluster health
description: Check REST server and cluster readiness, then read cluster- and node-level resource metrics.
api: openapi/quasar-rest-openapi-original.json
operations: [status-liveness, status-readiness, get-cluster, get-node]
---

# Monitor QuasarDB cluster health

Use the QuasarDB REST API to verify availability and read capacity/utilization for a time-series cluster.

## Steps

1. **Liveness.** Call `status-liveness` — `GET /api/status/liveness`. A 200 means the REST API server itself is up. (No auth required.)
2. **Readiness.** Call `status-readiness` — `GET /api/status/readiness`. A 200 means the REST server can reach the QuasarDB cluster; 500 means the cluster is unreachable.
3. **Authenticate** (see the run-query skill) and set `Authorization: Bearer <jwt>`.
4. **Cluster overview.** Call `get-cluster` — `GET /api/cluster` — for `{memoryTotal, memoryUsed, diskTotal, diskUsed, nodes[], status}`.
5. **Per-node drill-down.** For any node id from the cluster response, call `get-node` — `GET /api/cluster/nodes/{id}` — for CPU/memory/disk metrics and `quasardbVersion`. A 404 means the node id is unknown.

## Rules

- Poll liveness/readiness on a schedule for uptime checks; they are cheap and unauthenticated.
- Errors return `{"message": "<text>"}`; handle 400/404/500 per `errors/quasar-problem-types.yml`.
- For Prometheus-based monitoring, QuasarDB also exposes `prometheusRead`/`prometheusWrite` remote-read/write endpoints.
