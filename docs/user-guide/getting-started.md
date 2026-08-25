# Getting Started

This page gets you from "I have an account" to "I have a running GPU cluster"
in a few minutes.

## 1. Sign in

DC Suite is **single sign-on only** — there are no local passwords. Open your
installation's console URL and you'll be redirected to your organization's
identity provider (IdP). After you authenticate you land back in the console.

!!! note "Two doors"
    Most installations expose two sign-in paths: a general **user** login and
    a separate **operator/admin** login. Use whichever your administrator gave
    you. If you only need to run workloads, the user login is the one you want.

If sign-in fails, it's almost always one of: your account hasn't been added to
the organization yet, or your IdP session expired — retry, and if it persists,
contact an administrator.

## 2. Find your way around

Once signed in, the console groups everything into a few areas:

- **Clusters** — your GPU clusters: create, inspect, resize, delete.
- **Templates** — software stacks you can apply to a cluster.
- **Images** — golden machine images and their lifecycle.
- **Observability** — live metrics and logs for a chosen cluster.
- **Cost** — metered usage rolled up into money.
- **Bare-metals / Fleet** *(operators only)* — the physical machine pool.

## 3. Create your first cluster

1. Go to **Clusters → Create**.
2. Give it a name.
3. Choose a **hardware profile** (for example an L40S or H100 GPU node type).
   The picker shows how many nodes of that profile are **available now**.
4. Set the **node count** (bounded by what's available).
5. Pick an **image version** (default is the current `STABLE` image).
6. Submit. DC Suite reserves the machines, provisions them, and forms the
   cluster.

You'll see the cluster move through states — `PENDING → ALLOCATING →
PROVISIONING → CONFIGURING → READY`. When it reaches **READY**, it's yours to
use.

See [Creating a Cluster](creating-clusters.md) for every option, including
heterogeneous multi-profile clusters and init scripts.

## 4. Connect

Once the cluster is **READY**, open its detail page to find:

- **SSH access** — add your public key (or use the generated credentials) and
  connect to the login node.
- **Exposed ports** — any web UI you expose gets a stable public HTTPS URL.

See [Access & Networking](access-networking.md).

## 5. Run something

Submit a SLURM job from the console or the API, or apply a
[template](templates.md) (for example a scheduler or a framework stack) to
turn the cluster into a ready-to-use environment. See
[Running Jobs](jobs.md).

## 6. Clean up

When you're done, **delete** the cluster from its detail page. This releases
the machines back to the pool and stops metering. See
[Managing Clusters](managing-clusters.md).

---

**Next:** [Core Concepts](concepts.md) explains the nouns you just used —
clusters, profiles, images, tenants — so the rest of the guide makes sense.
