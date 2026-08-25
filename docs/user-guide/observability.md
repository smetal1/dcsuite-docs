# Observability

The **Observability** page gives you live metrics and streamed logs for a
chosen cluster, so you can see what your workload is doing without leaving the
console.

## Metrics

Pick a cluster and a time window. DC Suite charts the signals that matter for
GPU work:

- **GPU utilization** — per-GPU utilization across the cluster.
- **SLURM nodes by state** — how many nodes are idle, allocated, down.
- **SLURM jobs by state** — pending, running, completed.

Charts poll periodically and build a rolling window, so they fill in over the
first few polls after you open the page.

## Logs

The **Logs** panel streams control-plane logs for the selected cluster. You
can:

- Choose a **time window** (last 15 minutes, hour, or 6 hours).
- **Follow** the tail live, or pause to scroll back.
- **Refresh** on demand.

Logs are scoped to what you're allowed to see — you only get logs for clusters
within your access scope.

## When the backend isn't available

Observability depends on a metrics backend and a logs backend being wired up
by your operators. If they aren't:

- You'll see an informational note ("observability backend not configured")
  rather than charts — this is expected on installations that haven't enabled
  the stack.
- If logs show a backend error, it's an operator-side issue (backend
  unreachable or misconfigured); metrics may still work independently.

## API

For automation, the same data is available read-only:

- **`GET /v1/obs/metrics`** — instant metric queries.
- **`GET /v1/obs/logs`** — recent log lines for a cluster.
- **`GET /v1/obs/usage`** — usage counters (node-hours, GPU-hours) that also
  feed [Cost & Billing](cost.md).

---

**Next:** [Cost & Billing](cost.md).
