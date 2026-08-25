# Runbooks

Generic playbooks for the incidents operators hit most. Adapt the specifics to
your environment.

## "Only 0 nodes are available for &lt;profile&gt;" {#insufficient-capacity}

**Symptom:** the create form shows 0 available for a profile, or a create
fails with insufficient capacity.

**Cause:** availability counts pool nodes in the `AVAILABLE` state for that
profile. Zero means none are free — they're all in clusters, or stuck in a
non-available state (`QUARANTINE`, `SANITIZING`, `RESERVED`).

**Check:**

1. How many nodes of that profile exist, and what states are they in?
2. Are any stuck in `QUARANTINE`? → follow
   [A node is stuck QUARANTINE](#a-node-is-stuck-quarantine).
3. Are they legitimately all in running clusters? → that's real capacity
   pressure; return capacity or add machines.

## A node is stuck QUARANTINE {#a-node-is-stuck-quarantine}

**Symptom:** a machine that should be free sits in `QUARANTINE`; the fleet
sync won't recover it.

**How recovery is supposed to work:** the sync returns an unowned quarantined
node to `AVAILABLE` once the fleet manager certifies it healthy **and** its
bare-metal registration is `READY`.

**Common blockers:**

1. **Bare-metal row is `ERROR`.** Recovery is gated on `READY`, so an `ERROR`
   row pins the node in quarantine. Find out *why* the fleet manager reports
   the machine in error (below).
2. **Fleet manager stuck mid-terminate.** If a cluster was deleted while its
   machine was powered off, the fleet manager's release can stall: the
   machine's instance sits "terminating" because the post-teardown validation
   never ran, so the machine stays assigned and unhealthy.
3. **Validation keeps failing.** If the machine's in-band validation agent
   can't complete (repeated stale-timeout), the fleet manager parks the
   machine in error.

**Resolution (safest first):**

1. **Let the fleet manager finish on its own** — power the machine on so its
   validation completes; a healthy validation clears the error and the sync
   recovers the node.
2. If the fleet manager's controller is **wedged** (re-affirming a failed
   validation and not launching a new one), use the fleet manager's **own
   admin controls** to cancel the stale validation / force-release the
   machine. Prefer supported APIs over editing the fleet manager's state
   store directly — its controllers hold locks and versioned state, and raw
   edits risk inconsistency.

!!! warning
    Don't force a node to `AVAILABLE` in DC Suite while the fleet manager
    still refuses to allocate it — the next allocation will just fail. Fix the
    fleet-manager state first, then let the sync recover the node.

## Observability logs show a backend error {#observability-logs-error}

**Symptom:** the Observability **Logs** panel shows a backend error, while
**metrics** work.

**Common causes:**

1. **Missing tenant header.** A multi-tenant logs backend (auth enabled)
   rejects reads without a tenant header (HTTP 401), which surfaces as a 502
   "observability backend error." Set `observability.loki_tenant` to the org
   your log collector ingests under. (Metrics often work anyway because the
   metrics gateway injects a default org — hence logs fail but metrics don't.)
2. **Wrong stream selector.** If logs are labeled by your collector's scheme
   (e.g. `namespace`/`container`) rather than the default `service` label, the
   query matches nothing. Set `observability.log_stream_selector` to a
   selector that matches your control-plane pods; the server still appends its
   tenant/cluster scoping.
3. **Backend unhealthy.** If the logs backend itself is degraded (e.g. an
   ingester ring with too many unhealthy members), reads and writes fail
   independently of DC Suite. Restart/repair the backend; confirm its ring is
   healthy before retrying.

## A cluster is stuck in a transitional state

**Symptom:** a cluster sits in `ALLOCATING`/`PROVISIONING`/`CONFIGURING` far
longer than normal.

**Check:**

1. The operation's status and the workflow engine for the running workflow.
2. Whether the reserved nodes actually powered on and booted the image.
3. Node-level provisioning logs via the provisioning runner.

Because operations are durable workflows, a control-plane restart resumes them
rather than losing them. A genuinely failed provision lands the cluster in
`FAILED` with a reason on the detail page.

## Deploy didn't take effect

**Symptom:** a shipped fix isn't visible.

**Check:**

1. Did **both** the API server and the console roll to the new version? Many
   features need both — see [Deploys & Releases](deploys.md).
2. Is a config change waiting on a pod restart to be re-rendered?
3. Are the running images the expected tag (not a stale `latest`)?
