# Access & Networking

Once a cluster is **READY**, there are two ways to reach it: **SSH** to the
nodes, and **public HTTPS URLs** for any web port you expose.

## SSH access

Each cluster has a login user with a home directory and passwordless `sudo`
on the nodes. To connect:

1. **Add your public key.** From the cluster's Access panel (or
   `PUT /v1/clusters/{id}/ssh-keys`), register your SSH public key. The key
   set you send becomes the complete authorized list for the cluster.
2. **Connect** to the login node's address shown on the detail page:

    ```bash
    ssh <login-user>@<cluster-login-address>
    ```

!!! tip "Keys beat passwords"
    If you register one or more SSH keys, key-based auth is used and any
    default password is cleared. If you register no keys, some installations
    provision a default password for first access — rotate it by adding a key.

### Key delivery status

The Access panel reports whether your keys have actually **landed on the
nodes**, not just that they were accepted by the control plane:

- **Keys applied** — delivered and confirmed on the node.
- **Applying / pending** — delivered, waiting for the node agent to apply.
- **Node agent can't apply keys** — the node needs attention (a reimage may be
  required). Tell an operator.

This distinction exists so a node silently failing to apply keys doesn't look
like success.

## Exposing web UIs (HTTP port ingress)

Many workloads run a web UI — a notebook server, a dashboard, a training UI.
DC Suite gives **any port you expose a stable, per-cluster public HTTPS URL**,
so you don't fight with node IPs or SSH tunnels.

1. In the cluster's Access/Ports panel, **expose a port** (the port your app
   listens on inside the cluster).
2. DC Suite assigns a stable hostname like
   `https://<name>-<clusterid>.<your-domain>` and routes it to that port over
   TLS.
3. Open the URL — your app's UI comes up directly.

### Restricting and disabling access

Each exposed port has controls:

- **Source restriction** — limit who can reach the URL to a set of source IP
  ranges (CIDR). Anything outside is refused.
- **Disable public exposure** — turn the public URL off entirely while keeping
  the port defined, then re-enable when you need it.

!!! note "Ports are freed on delete"
    When a cluster is deleted, the ports it held are released back to the
    available range for reuse by future clusters.

---

**Next:** [Running Jobs](jobs.md).
