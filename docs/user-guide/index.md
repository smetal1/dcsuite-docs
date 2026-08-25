# User Guide

This guide is for people who **use** DC Suite to run GPU workloads — data
scientists, ML engineers, researchers, and their teams. You do not need to
know how the platform is deployed; you just need an account.

## What you can do

| Task | Where |
| --- | --- |
| Sign in and find your way around | [Getting Started](getting-started.md) |
| Understand clusters, profiles, images, tenants | [Core Concepts](concepts.md) |
| Spin up a GPU cluster | [Creating a Cluster](creating-clusters.md) |
| Resize, power-cycle, or delete a cluster | [Managing Clusters](managing-clusters.md) |
| SSH in and expose web UIs | [Access & Networking](access-networking.md) |
| Understand storage tiers and keep your data | [Storage & Data](storage.md) |
| Submit and track SLURM jobs | [Running Jobs](jobs.md) |
| Apply software stacks to a cluster | [Templates](templates.md) |
| Build and promote machine images | [Images](images.md) |
| Watch live metrics and logs | [Observability](observability.md) |
| See what you are spending | [Cost & Billing](cost.md) |
| Understand quotas and availability | [Quotas & Limits](quotas.md) |
| Understand isolation and access control | [Security & Isolation](security.md) |
| Automate against the REST API | [API Reference](api.md) |

## Prerequisites

- An account in your organization's DC Suite installation (created by an
  administrator and signed in via your identity provider).
- A role that grants the actions you want to perform. If a button is missing
  or an action is refused, you likely lack the permission — ask an admin.
- For SSH access: an SSH key pair on your machine.
