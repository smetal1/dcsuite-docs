# Glossary

**Bare-metal machine**
: A physical server registered with DC Suite (spec + BMC coordinates) that,
  once onboarded, becomes a node in the pool.

**BMC**
: Baseboard Management Controller — a server's out-of-band management
  interface (Redfish/IPMI) used for power control and health, independent of
  the host OS.

**Burn-in**
: A validation pass over a freshly built image (GPU present, scheduler works,
  interconnect health) that must pass before the image is trusted.

**Cluster**
: A set of nodes DC Suite reserves, provisions, and wires together for a user.
  Has a lifecycle from `PENDING` to `READY` to `DELETED`.

**Control plane**
: The DC Suite services (API server, console, workflow engine, provisioning
  runner, database) that orchestrate everything.

**Fleet manager**
: The system that owns the physical machines — discovery, power, validation,
  allocation. DC Suite drives it; it drives the hardware.

**Golden image**
: A versioned, immutable machine image nodes boot from. Moves through
  `BUILT → CANARY → STABLE → DEPRECATED`.

**Hardware profile**
: A class of machine (GPU model, GPUs per node, interconnect), e.g.
  `gpu-l40s`. Clusters are built from one or more profiles.

**Node**
: One machine in a cluster. In the fleet it has states `AVAILABLE`,
  `RESERVED`, `IN_CLUSTER`, `SANITIZING`, `QUARANTINE`.

**Node group**
: A set of nodes of one profile within a cluster. Multiple groups make a
  heterogeneous cluster.

**Operation**
: An asynchronous, durable action (create/resize/delete) with its own status,
  driven by the workflow engine.

**Organization**
: The top-level account boundary. Cost and access are scoped by organization.

**Quarantine**
: A node state meaning "do not hand this out" — used when the fleet manager
  reports a machine assigned/sanitizing/unhealthy, so it's never allocated by
  mistake.

**Sanitize**
: The wipe + re-validation a machine goes through when released from a cluster
  before it returns to `AVAILABLE`.

**Scoped RBAC**
: Role-based access control where a role's permissions apply at a specific
  scope — installation, organization, or tenant.

**SSO**
: Single sign-on. All DC Suite sign-in goes through an OIDC identity provider;
  there are no local passwords.

**Template (stack)**
: A packaged piece of software applied to a running cluster (scheduler,
  framework, dashboard). Global (admin-published) or private (user-owned).

**Tenant**
: A project/team within an organization, with its own quotas and clusters.

**X-Scope-OrgID**
: The tenant header multi-tenant metrics/logs backends require on reads;
  configured via `loki_tenant` / `prometheus_tenant`.
