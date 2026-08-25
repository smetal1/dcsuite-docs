# Installation

DC Suite runs on Kubernetes. This page covers the shape of an install; adapt
the specifics to your environment.

## Prerequisites

- A **Kubernetes cluster** to host the control plane.
- **PostgreSQL** (managed or in-cluster) for the system of record.
- A **workflow engine** endpoint for durable operations.
- A reachable **fleet manager** managing your bare-metal pool.
- An **identity provider** for SSO.
- *(Optional)* metrics and logs backends for observability.
- An **ingress controller** and wildcard DNS for the console and for the
  per-cluster public URLs DC Suite mints.
- A **container registry** the cluster can pull the DC Suite images from.

## Components to deploy

| Workload | Notes |
| --- | --- |
| API server | The control plane. Needs DB, workflow, fleet-manager, SSO config. |
| Web console | Static SPA served behind the same domain. |
| Provisioning runner | Runs per-cluster infrastructure jobs. |
| PostgreSQL | If not using a managed database. |

## Install with Helm

A Helm chart packages the workloads and their configuration. At a high level:

```bash
helm upgrade --install dc-suite ./deploy/helm/dc-suite \
  --namespace dc-suite --create-namespace \
  --values my-values.yaml
```

Your `values.yaml` supplies environment (`prod`), the backend endpoints, SSO
settings, and image tags. See [Configuration](configuration.md) for the keys.

## Ingress and DNS

DC Suite needs two kinds of routable names:

1. The **console** hostname (e.g. `dc-suite.example.com`).
2. A **wildcard** for per-cluster public URLs (e.g. `*.clusters.example.com`),
   so each exposed cluster port gets a stable HTTPS hostname with a
   certificate.

Point the wildcard at your ingress controller and give it a certificate
resolver (for example ACME) so new hostnames get certificates automatically.

## Bring machines into the pool

Deploying the control plane doesn't give you capacity — you still need to
onboard physical machines through the fleet manager. See
[Bare-Metal Onboarding](bare-metal.md).

## Verify

- The console loads and redirects to SSO.
- `GET /healthz` returns OK.
- `GET /v1/profiles` lists your hardware profiles with availability once
  machines are onboarded.
- A test cluster reaches `READY` and can be deleted cleanly.

---

**Next:** [Configuration](configuration.md).
