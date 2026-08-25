# FAQ & Troubleshooting

## Using DC Suite

**Why can't I sign in?**
: DC Suite is SSO-only. Sign-in fails if your account isn't yet in an
  organization, or your IdP session expired. Retry; if it persists, ask an
  administrator to confirm your account and role.

**A button or page is missing — why?**
: Controls are gated by your role at your scope. If you can't see or do
  something, you likely lack that permission. Ask an admin to grant it.

**The profile shows "0 nodes available." What do I do?**
: The fleet currently has no free machines of that profile. Try a smaller node
  count, a different profile, or wait for machines to return to the pool. If
  you believe machines should be free, an operator may need to recover a
  stuck node — see the [capacity runbook](../operators/runbooks.md#insufficient-capacity).

**My cluster went to `FAILED`. Now what?**
: Open the cluster's detail page — the failure reason is shown. Common causes
  are insufficient capacity (lower the count) or a node failing to boot the
  image (retry or pin a different image version).

**I added my SSH key but can't connect.**
: Check the Access panel's key status. "Applying/pending" means the node
  hasn't applied it yet — wait a moment. "Node agent can't apply keys" means
  the node needs operator attention.

**My exposed web UI URL doesn't load.**
: Confirm the port you exposed is the one your app actually listens on, that
  public exposure isn't disabled, and that your source IP isn't excluded by a
  source restriction. DNS for new hostnames can take a moment to resolve.

**The Observability logs panel shows an error.**
: That's an operator-side backend issue, not something you can fix. Metrics
  may still work. Let your operators know — see the
  [logs runbook](../operators/runbooks.md#observability-logs-error).

**My cost report shows "unrated hours."**
: That's real usage with no price attached because a rate card is missing for
  that profile. Ask an administrator to add one.

## Operating DC Suite

**Do I have to deploy the server and console together?**
: When a change touches both, yes — otherwise the UI may call an endpoint the
  server doesn't have yet (or vice versa). See
  [Deploys & Releases](../operators/deploys.md).

**A machine shows `ERROR` but it's in a running cluster. Is it broken?**
: Usually not. It's typically a stale background validation the fleet couldn't
  run while the machine was busy. DC Suite displays such machines as **IN_USE**
  rather than a red error. If it's genuinely idle and stuck, see the
  [node recovery runbook](../operators/runbooks.md#a-node-is-stuck-quarantine).

**Metrics work but logs don't. Why just logs?**
: Multi-tenant logs backends require a tenant header on reads that the metrics
  gateway often injects automatically for metrics. Set
  `observability.loki_tenant`. See the
  [logs runbook](../operators/runbooks.md#observability-logs-error).

**Does the control plane restarting lose in-flight cluster operations?**
: No. Operations run as durable workflows and resume after a restart.

## Concepts

**What's the difference between a cluster node's state and a bare-metal
machine's state?**
: A **node** state (`AVAILABLE`, `IN_CLUSTER`, `QUARANTINE`, …) is the fleet's
  view used for scheduling. A **bare-metal** state (`REGISTERED`, `READY`,
  `ERROR`, …) is the operator's onboarding record for the physical machine.
  They're linked but distinct.

**What's the difference between a template and an image?**
: An **image** is the base OS/machine image nodes boot from. A **template**
  (stack) is software you apply *on top* of a running cluster.

**Can one cluster mix GPU types?**
: Yes — use multiple node groups, one per profile. That's a heterogeneous
  cluster.
