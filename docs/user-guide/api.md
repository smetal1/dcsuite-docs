# API Reference

Everything the console does is backed by a REST API under `/v1`. Use it to
automate cluster lifecycle, jobs, templates, images, and reporting.

## Base URL and versioning

```
https://your-dc-suite.example.com/v1
```

All endpoints live under the `/v1` prefix. A public `GET /healthz` (no auth)
reports liveness.

## Authentication

Send a bearer token:

```
Authorization: Bearer <token>
```

Tokens are issued through your installation's SSO/identity integration. Every
request is authorized against your **scoped role** — you can only act on
resources within your organization/tenant scope. `GET /v1/whoami` returns your
identity, roles, and effective scope.

## Conventions

- **JSON** request and response bodies.
- **Async operations.** Create/resize/delete return an `operation_id`; poll
  `GET /v1/clusters/{id}/operations/{opID}` until it completes.
- **Errors** use standard HTTP status codes with a JSON `{"error": "..."}`
  body. `403` means your role lacks the permission; `404` means not found (or,
  for some endpoints, not accessible to your scope).

## Status codes

| Code | Meaning | What to do |
| --- | --- | --- |
| `200` / `201` | Success | — |
| `202` | Accepted — async action started | Poll the operation. |
| `400` | Bad request (validation) | Fix the request body/params. |
| `401` | Not authenticated | Refresh/attach a valid bearer token. |
| `403` | Authenticated but not permitted | Your role lacks the permission at this scope. |
| `404` | Not found (or not visible to your scope) | Check the id and your scope. |
| `409` | Conflict (e.g. state doesn't allow the action) | Re-read the resource; retry when valid. |
| `429` | Rate limited | Back off and retry with a delay. |
| `5xx` | Server error | Retry with backoff; if persistent, contact operators. |

Error bodies are JSON: `{"error": "human-readable reason"}`.

## Operations (async actions)

Create, resize, delete, and power actions are asynchronous. The initial call
returns an `operation_id`; poll:

```
GET /v1/clusters/{id}/operations/{opID}
```

`status` moves through `RUNNING → COMPLETED` or `RUNNING → FAILED` (with a
reason). Poll at a modest interval (≈10s). Operations are durable — a
control-plane restart resumes them, so a transient poll error doesn't mean the
work stopped.

## Pagination & filtering

List endpoints return arrays; where a collection can be large, use the
endpoint's documented `page`/`per_page` (or cursor) parameters and follow
`result_info` to page through. Treat unknown response fields as
forward-compatible — ignore what you don't use rather than failing on it.

## Endpoint map

### Identity

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v1/whoami` | Current identity, roles, scope |
| GET | `/healthz` | Liveness (unauthenticated) |

### Clusters

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v1/clusters` | Create a cluster |
| GET | `/v1/clusters` | List clusters |
| GET | `/v1/clusters/{id}` | Get a cluster |
| DELETE | `/v1/clusters/{id}` | Delete a cluster |
| GET | `/v1/clusters/{id}/nodes` | List a cluster's nodes |
| GET | `/v1/clusters/{id}/operations/{opID}` | Poll an operation |
| PUT | `/v1/clusters/{id}/ssh-keys` | Replace authorized SSH keys |
| POST | `/v1/clusters/{id}/power` | Power action across nodes |
| GET | `/v1/clusters/{id}/power` | Per-node live power state |
| POST | `/v1/clusters/{id}/nodes/{nodeId}/power` | Power action on one node |

### Jobs

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/v1/clusters/{id}/jobs` | Submit a SLURM job |
| GET | `/v1/clusters/{id}/jobs` | List jobs |
| GET | `/v1/clusters/{id}/jobs/{jobID}` | Get a job |
| DELETE | `/v1/clusters/{id}/jobs/{jobID}` | Cancel a job |

### Fleet & profiles

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v1/profiles` | Hardware profiles + live availability |

### Templates (stacks)

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v1/stacks` | List templates (scoped by visibility) |
| GET | `/v1/stacks/{name}` | Get a template |
| GET | `/v1/stacks/{name}/download` | Download the template `.zip` |
| POST | `/v1/stacks` | Add a template (zip or URL) |
| PATCH | `/v1/stacks/{name}` | Update (e.g. icon) |
| DELETE | `/v1/stacks/{name}` | Delete a custom template |

### Images

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v1/images` | List images |
| GET | `/v1/images/{name}/{version}/burnin` | Burn-in report |
| POST | `/v1/images/{name}/{version}/promote` | Advance lifecycle |
| POST | `/v1/images/{name}/{version}/deprecate` | Deprecate |

### SSH keys

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v1/ssh-keys` | List your keys |
| POST | `/v1/ssh-keys` | Add a key |
| DELETE | `/v1/ssh-keys/{id}` | Remove a key |

### Observability

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v1/obs/metrics` | Instant metric query |
| GET | `/v1/obs/logs` | Recent cluster logs |
| GET | `/v1/obs/usage` | Usage counters |

### Cost & billing

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/v1/billing/costs` | Cost report (JSON) |
| GET | `/v1/billing/costs.csv` | Cost report (CSV) |
| GET | `/v1/billing/currencies` | Currency formatting table |
| PATCH | `/v1/orgs/{id}/billing-currency` | Set an org's billing currency |

## Example: full create → wait → use

```bash
BASE=https://your-dc-suite.example.com
AUTH="Authorization: Bearer $DCS_TOKEN"

# 1. Create
CREATE=$(curl -sS -X POST "$BASE/v1/clusters" -H "$AUTH" \
  -H "Content-Type: application/json" \
  -d '{"name":"demo","profile":"gpu-l40s","node_count":1}')
CID=$(echo "$CREATE" | jq -r .cluster.id)
OP=$(echo "$CREATE" | jq -r .operation_id)

# 2. Poll the operation until it finishes
while :; do
  STATE=$(curl -sS "$BASE/v1/clusters/$CID/operations/$OP" -H "$AUTH" | jq -r .status)
  echo "op: $STATE"; [ "$STATE" = "COMPLETED" ] && break
  [ "$STATE" = "FAILED" ] && { echo "create failed"; exit 1; }
  sleep 10
done

# 3. Submit a job
curl -sS -X POST "$BASE/v1/clusters/$CID/jobs" -H "$AUTH" \
  -H "Content-Type: application/json" \
  -d '{"script":"#!/bin/bash\n#SBATCH --gpus=1\nsrun nvidia-smi\n"}'
```

> Endpoint paths above are stable; exact request/response schemas may add
> fields over time. Treat unknown fields as forward-compatible and ignore them
> if you don't use them.
