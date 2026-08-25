# Configuration

DC Suite reads a single YAML config (rendered into the API server) plus
secrets. This page describes the meaningful knobs. Exact key names may vary by
version; treat this as the map.

## Environment

```yaml
env: prod          # demo | prod
listen_addr: ":8443"
```

- **`demo`** — self-contained defaults, stdout exporters, no external
  dependencies. For local/dev.
- **`prod`** — **fails closed**: the server refuses to start unless the
  required endpoints below are set.

## Database

```yaml
database_url: "postgres://USER:${DB_PASSWORD}@HOST:5432/DBNAME?sslmode=disable"
```

The password is substituted from a secret at render time — never commit it.

## Workflow engine

```yaml
workflow_host_port: "workflow-frontend.internal:7233"
workflow_namespace: "dc-suite"
```

Point at your durable workflow engine. A dedicated namespace keeps DC Suite's
workflows isolated with their own retention.

## Fleet manager

```yaml
fleet:
  mode: real           # real | fake (tests/demo)
  # endpoint + auth for the fleet manager's API
```

The `fake` mode is an in-memory fleet for tests and demos; `real` drives the
actual fleet manager.

## Observability backends

```yaml
observability:
  otlp_endpoint:  "http://otel-collector.monitoring:4318"
  prometheus_url: "http://metrics-gateway.monitoring/prometheus"
  loki_url:       "http://logs-gateway.monitoring"
  # Multi-tenant log/metric backends require a tenant header on reads:
  loki_tenant:       ""   # X-Scope-OrgID for a multi-tenant logs backend
  prometheus_tenant: ""   # X-Scope-OrgID for a multi-tenant metrics backend
  # Loki stream selector for /obs/logs (defaults to a forge-native selector):
  log_stream_selector: ""
```

!!! warning "Multi-tenant log/metric backends need a tenant header"
    If your logs backend runs with authentication enabled (multi-tenant), it
    rejects reads that carry no tenant header — which surfaces to users as an
    "observability backend error." Set `loki_tenant` to the org your log
    collector ingests under. Metrics backends whose query gateway injects a
    default org need no `prometheus_tenant`.

!!! note "Log stream selector"
    The default log selector assumes logs are labeled by a `service` stream
    label. If your log collector labels streams differently (e.g. by
    `namespace`/`container`), set `log_stream_selector` to a selector that
    matches your control-plane pods. The server always appends its own
    tenant/cluster scoping filters after your selector, so isolation is
    preserved. See the [Observability runbook](runbooks.md#observability-logs-error).

Runtime is **fail-open** on these endpoints: if one is wrong or unreachable,
provisioning keeps working and only the affected observability read degrades.

## Identity / SSO

Configure your OIDC identity provider (issuer, client id, client secret via a
secret). See [SSO & Identity](sso.md).

## Image tags

Pin the DC Suite component images to immutable version tags in your Helm
values so a deploy is reproducible and you always know what's running. Avoid
tracking a moving `:latest` tag in production.

## Secrets

Keep credentials in your secret manager, injected at render time:

- Database password
- Fleet manager / SSO client secrets
- Backend credentials (if your metrics/logs backends require basic auth)

Never place secrets in the YAML config or in this documentation.

---

**Next:** [Bare-Metal Onboarding](bare-metal.md).
