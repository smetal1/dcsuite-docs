# Deploys & Releases

How new versions of DC Suite are built and shipped.

## Build pipeline

Releases are built by CI, not from a workstation. A tag push (`vX.Y.Z`)
triggers a pipeline that:

1. **Verifies** — builds, vets, and runs the full test suite against a fresh
   database. A tag that fails verification never publishes an image.
2. **Builds and publishes** the component images (API server, console,
   provisioning runner) to your registry, tagged with the release version and
   `latest`.

Pin deployments to the **immutable version tag**, not `latest`, so you always
know what's running.

## Deploying a release

Deployment updates the running workloads to the new image tags. Depending on
your setup this is either:

- **GitOps** — a reconciler watches the registry/repo and rolls the new tag
  out automatically, or
- **Manual** — an operator updates the image tags (Helm values or a direct
  image update) and lets Kubernetes roll the deployment.

Either way, the two things that must move together are:

!!! warning "Deploy the server and the console together"
    Many features span **both** the API server and the console. Shipping only
    the console (or only the server) can leave the UI calling an endpoint the
    server doesn't have yet — or vice versa. When a change touches both, roll
    both to the same version.

## Config vs. code

- **Code changes** ship as new images (a new version tag).
- **Config changes** (endpoints, tenant headers, selectors) ship as updates to
  the rendered config / Helm values and take effect on the next pod restart.

When a change needs both (new config keys read by a new binary), apply the
config first (older binaries ignore unknown keys), then roll the new image so
it picks the config up.

## Verifying a deploy

- Confirm the workloads report the expected image tag and are `Ready`.
- Smoke-test the critical path: sign in, list profiles, create and delete a
  small cluster.
- Watch logs for config errors on startup (a `prod` server fails closed on
  missing required config).

## Rollback

Because images are immutable and pinned, rollback is redeploying the previous
version tag. Keep the last known-good tag handy. Database migrations should be
backward-compatible within a release window so a rollback doesn't strand the
schema.

---

**Next:** [Runbooks](runbooks.md).
